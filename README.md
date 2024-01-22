# Deep NLP: Literature Analysis

## Note

This project is still in its final stages of development. However, most individual modules are complete and can be used or repurposed. The documentation is also comprehensive and mostly up-to-date. However, please know that some of the organization and in-project README.txt files have not been updated yet. For example, the README.txt in the **[Archive](https://github.com/kkrishna2023/deep_nlp_on_sf_literature/tree/main/Archive)** directory does not entirely capture how the directory has evolved since then. 

As of this writing, I am currently in the process of [fine-tuning a RoBERTa model](https://colab.research.google.com/drive/1Md5Lpe4WiLxeDUzxZk8vdkRmA50MIUMh?usp=sharing) with the output acquired from `GPT_NER_Round_1.py`, in preparation for performing last-stage NER using [RoBERTa](https://github.com/facebookresearch/fairseq/blob/main/examples/roberta/README.md). If you wish to explore the project code, I suggest beginning with `CorpusProcessor_with_NER.py` in **[main files](https://github.com/kkrishna2023/deep_nlp_on_sf_literature/tree/main/main%20files)** (which actually does not contain any NER implementation within itself yet, despite being fully functional otherwise). There you will find code for multifarious data preprocessing. If you wish to run it as a script, however, please set the `FILEPATH` variable to a file you have access to and want to process. I have not hosted "internet_archive_scifi_v3.txt" (the corpus file I was using) on GitHub yet, thanks to its size.

## Overview and guide

Most of the action as of the current version is contained in the directories **[main files](https://github.com/kkrishna2023/deep_nlp_on_sf_literature/tree/main/main%20files)**, **[TF-IDF](https://github.com/kkrishna2023/deep_nlp_on_sf_literature/tree/main/TF-IDF)**, and **[LLM](https://github.com/kkrishna2023/deep_nlp_on_sf_literature/tree/main/LLM)**. In **main files**, begin with `CorpusProcessor_with_NER.py` that was used for preprocessing the corpus. Then go to `NER.py` that performs first-stage NER on the corpus using [spaCy](https://github.com/explosion/spaCy) and stores the output in "NER_output2.csv" by default.

After performing basic NER, you can check out `sentences_of_entities.py`. If you run it as a script, it (attempts to) extract the famed 3.5M sentences from "sci-fi_text_cleaned.csv", the csv file representing a [Pandas dataframe](https://github.com/pandas-dev/pandas) that contains the corpus as a list of sentences with their cleaned versions. Then from this list, `sentences_of_entities` extracts all the sentences that contain any of the entities that were found by spaCy's NER. (Note that these entities are brought in from "entity_list.txt" which is a file I created using a separate script, after running `NER.py`. "entity_list.txt" _is_ small enough to be included in the "main files" directory, so it is. Also note that in the current version of the code, "sci-fi_text_cleaned.csv" has not been added already to **main files**. You may have to create this file or slightly modify the code in `sentences_of_entities` to have it work as a script)

Now the results of `sentences_of_entities.py` will be stored in the **LLM** directory, in "sents_containing_named_entities.txt". But don't go to the **LLM** directory just yet: first go to **TF-IDF** and run `TF-IDF.py` as a script. This will use [scikit-learn](https://github.com/scikit-learn/scikit-learn)'s [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html) and KMeans clustering to extract a **diverse** batch of 8000 sentences from `sents_containing_named_entities.txt`. The diverse batch of sentences will be stored in `representative_sents_for_GPT_labelling.txt` that remains in the `TF-IDF` directory. Why shall we be using a representative sample rather than the full 114,000+ entity-containing sentences? Because otherwise, we'll need a _lot_ of [OpenAI API](https://github.com/openai/openai-python) tokens to run a round of custom NER using [GPT-3.5 Turbo](https://platform.openai.com/docs/guides/text-generation/chat-completions-api).

Finally, you can now go to the directory **LLM**, open `GPT_NER_round_1.py`, and run it as a script. I have already created data you can use, so when asked about it you can use any key except "Y" to generate RoBERTa training data. If you want to see real magic however, first go to `api_key.py` and insert a valid OpenAI API key. Then run `GPT_NER_round_1` again and say "Y" when it asks you for input. You can then see as we use [spaCyLLM](https://github.com/explosion/spacy-llm) and [OpenAI API](https://github.com/openai/openai-python) to label and extract terms from the text as **TECHNOLOGY**, **CONCEPT**, or **MISCELLANEOUS SIGNIFICANT**. Then, we [tokenize](https://github.com/huggingface/transformers/blob/main/src/transformers/models/roberta/tokenization_roberta.py) this output appropriately and prepare to use it for [fine-tuning a RoBERTa model](https://colab.research.google.com/drive/1Md5Lpe4WiLxeDUzxZk8vdkRmA50MIUMh?usp=sharing), so RoBERTa can perform the remainder of our required NER - for free.

**Note on 'LLM'**: `LLM_Interactor.py` is the module that handles the creation of - you guessed it - an LLM interaction object. It configures spaCyLLM - [please check the documentation](https://spacy.io/api/large-language-models) - using the configuration strings residing in `config_strings_2.py`, which in turn refers to `fewshot.json` for its [fewshot-training](https://blog.paperspace.com/few-shot-learning/) info. **This means that `LLM_Interactor` as well as `GPT_NER_round_1` are very independent and malleable modules.** By modifying `fewshot.json`, you can train GPT-3.5 to perform NER for any other kind of domain-specific task. You can also change the spacyLLM configuration by modifying `config_strings_2.py`, where you can modify the "labels" to be fed for your own domain-specific task. For example, you can change these lines:

  `[components.llm.task]`
  `@llm_tasks = "spacy.NER.v3"`
  `labels = ["TECHNOLOGY", "CONCEPT", "MISCELLANEOUS SIGNIFICANT"]`

to these lines:

  `[components.llm.task]`
  `@llm_tasks = "spacy.NER.v3"`
  `labels = ["FINANCIAL ENTITY", "NON-FINANCIAL ENTITY"]`

if you were doing NLP on finance-related texts.

You can also change the LLM model used to other ones supported by spaCyLLM.
