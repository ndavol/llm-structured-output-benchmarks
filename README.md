# 🧩 LLM Structured Output Benchmarks


## 🏆 Benchmark Results [2024-06-22]
1. Multi-label classification
    | Framework                                                                                           |                 Model                | Reliability |
    |-----------------------------------------------------------------------------------------------------|:------------------------------------:|:-----------:|
    | [Mirascope](https://github.com/mirascope/mirascope)                                                 |          gpt-3.5-turbo-0125          |    1.000    |
    | [Fructose](https://github.com/bananaml/fructose)                                                    |          gpt-3.5-turbo-0125          |    1.000    |
    | [Outlines](https://github.com/outlines-dev/outlines)                                                | unsloth/llama-3-8b-Instruct-bnb-4bit |    1.000    |
    | [Llamaindex](https://docs.llamaindex.ai/en/stable/examples/output_parsing/openai_pydantic_program/) |          gpt-3.5-turbo-0125          |    0.999    |
    | [Instructor](https://github.com/jxnl/instructor)                                                    |          gpt-3.5-turbo-0125          |    0.994    |
    | [Marvin](https://github.com/PrefectHQ/marvin)                                                       |          gpt-3.5-turbo-0125          |    0.975    |

## 🧪 Benchmark methodology
1. Multi-label classification:
    - **Task**: Given a text, predict the labels associated with it.
    - **Data**:
        - Base data: [Alexa intent detection dataset](https://huggingface.co/datasets/AmazonScience/massive)
        - Benchmarking test is run using synthetic data generated by running: `python -m data_sources.generate_dataset generate-multilabel-data`.
        - The synthetic data is generated by sampling and combining rows from the base data to achieve multiple classes per row according to some distribution for num classes per row. See `python -m data_sources.generate_dataset generate-multilabel-data --help` for more details.
    - **Prompt**: `"Classify the following text: {text}"`
    - **Evaluation Metrics**: 
        1. Reliability: The percentage of times the framework returns valid labels without errors.
    - **Experiment Details**: Run each row through the framework `n_runs` number of times and log the percent of successful runs for each row. Reliability is the average of all the rows `percent_successful` values.
1. Named Entity Recognition (🚧 Coming soon)

## 🏃 Run the benchmark
1. Install the requirements using `pip install -r requirements.txt`
1. Set the OpenAI api key: `export OPENAI_API_KEY=sk-...`
1. Run the benchmark using `python -m main run-benchmark`
1. Raw results are stored in the `results` directory.
1. Generate the results using `python -m main generate-results`
1. To get help on the command line arguments, add `--help` after the command. Eg., `python -m main run-benchmark --help`

## 📊 Adding new data
1. Create a new pandas dataframe pickle file with the following columns:
    - `text`: The text to be sent to the framework
    - `labels`: List of labels associated with the text
    - See `data/multilabel_classification.pkl` for an example.
1. Add the path to the new pickle file in the `./config.yaml` file under the `source_data_pickle_path` key for all the frameworks you want to test.
1. Run the benchmark using `python -m main run-benchmark` to test the new data on all the frameworks!
1. Generate the results using `python -m main generate-results`

## 🏗️ Adding a new framework
The easiest way to create a new framework is to reference the `./frameworks/instructor_framework.py` file. Detailed steps are as follows:

1. Create a .py file in frameworks directory with the name of the framework. Eg., `instructor_framework.py` for the instructor framework.
1. In this .py file create a class that inherits `BaseFramework` from `frameworks.base`.
1. The class should define an `init` method that initializes the base class. Here are the arguments the base class expects:
    - `name` (str): name of the task that the framework is being tested on. Obtained from `./config.yaml` file. Eg., `"multilabel_classification"`
    - `prompt` (str): Prompt template used. Obtained from the `init_kwargs` in the `./config.yaml` file. 
    - `llm_model` (str): LLM model to be used. Obtained from the `init_kwargs` in the `./config.yaml` file.
    - `llm_model_family` (str): LLM model family to be used. Current supported values as `"openai"` and `"transformers"`. Obtained from the `init_kwargs` in the `./config.yaml` file.
    - `retries` (int): Number of retries for the framework. Default is $0$. Obtained from the `init_kwargs` in the `./config.yaml` file.
    - `source_data_picke_path` (str): Path to the source data pickle file. Obtained from the `init_kwargs` in the `./config.yaml` file.
    - `sample_rows` (int): Number of rows to sample from the source data. Useful for testing on a smaller subset of data. Default is $0$ which uses all rows in source_data_pickle_path for the benchmarking. Obtained from the `init_kwargs` in the `./config.yaml` file.
    - `response_model` (Any): The response model to be used. Internally passed by the benchmarking script.
1. The class should define a `run` method that takes three arguments:
    1. `inputs`: a dictionary of `{"text": str}` where `str` is the text to be sent to the framework
    1. `n_runs`: number of times to repeat each text
    1. `expected_response`: Output expected from the framework
1. This `run` method should create another `run_experiment` function that takes `inputs` as argument, runs that input through the framework and returns the output.
1. The `run_experiment` function should be annotated with the `@experiment` decorator from `frameworks.base` with `n_runs` and `expected_resposne` as arguments.
1. The `run` method should call the `run_experiment` function and return the three outputs `predictions`, `percent_successful` and `accuracy`.
1. Import this new class in `frameworks/__init__.py`.
1. Add a new entry in the `./config.yaml` file with the name of the class as the key. The yaml entry can have the following fields
    - `name`: name of the task that the framework is being tested on. Obtained from `./config.yaml` file. Eg., `"multilabel_classification"`
    - `n_runs`: number of times to repeat each text
    - `init_kwargs`: all the arguments that need to be passed to the `init` method of the class, including those mentioned in step 3 above.

## 🧭 Roadmap
1. Framework related tasks:
    | Framework                                                                                           | Multi-label classification | Named Entity Recognition | Synthetic Data Generation |
    |-----------------------------------------------------------------------------------------------------|:--------------------------:|:------------------------:|:-------------------------:|
    | [Instructor](https://github.com/jxnl/instructor)                                                    |          ✅ OpenAI          |        💭 Planning        |         💭 Planning        |
    | [Mirascope](https://github.com/mirascope/mirascope)                                                 |          ✅ OpenAI          |        💭 Planning        |         💭 Planning        |
    | [Fructose](https://github.com/bananaml/fructose)                                                    |          ✅ OpenAI          |        💭 Planning        |         💭 Planning        |
    | [Marvin](https://github.com/PrefectHQ/marvin)                                                       |          ✅ OpenAI          |        💭 Planning        |         💭 Planning        |
    | [Llamaindex](https://docs.llamaindex.ai/en/stable/examples/output_parsing/openai_pydantic_program/) |          ✅ OpenAI          |        💭 Planning        |         💭 Planning        |
    | [Outlines](https://github.com/outlines-dev/outlines)                                                |      ✅ HF Transformers     |        💭 Planning        |         💭 Planning        |
    | [LLM format enforcer](https://github.com/noamgat/lm-format-enforcer)                                |        🚧 In Progress       |        💭 Planning        |         💭 Planning        |
    | [Jsonformer](https://github.com/1rgs/jsonformer)                                                    |        🚧 In Progress       |        💭 Planning        |         💭 Planning        |
    | [Strictjson](https://github.com/tanchongmin/strictjson)                                             |        🚧 In Progress       |        💭 Planning        |         💭 Planning        |
    | [Guidance](https://github.com/guidance-ai/guidance)                                                 |        🚧 In Progress       |        💭 Planning        |         💭 Planning        |
    | [DsPy](https://dspy-docs.vercel.app/docs/building-blocks/typed_predictors)                          |        🚧 In Progress       |        💭 Planning        |         💭 Planning        |
    | [Langchain](https://python.langchain.com/v0.2/docs/tutorials/extraction/)                           |        🚧 In Progress       |        💭 Planning        |         💭 Planning        |
1. Others
    - CICD pipeline for benchmark run automation
    - Async run

## 💡 Contribution guidelines
Contributions are welcome! Here are the steps to contribute:
1. Please open an issue with any new framework you would like to add. This will help avoid duplication of effort.
1. Once the issue is assigned to you, pls submit a PR with the new framework!

## 🎓 Citation
To cite LLM Structured Output Benchmarks in your work, please use the following bibtex reference:
```
```

## 🙏 Feedback
If this work helped you in any way, please consider ⭐ this repository to give me feedback so I can spend more time on this project.
