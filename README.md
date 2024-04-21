# LLM Load Testing

# Goal

Find out how many http requests per second the LLM(Large Language Model) service can handle.

# Background

Large Language Models (LLMs) have become an integral part of modern applications.
Unlike traditional databases, LLMs are known for slower response times.
While building a solution on top of the LLM service, we need to know how many requests per second the service can
handle.
So then we can design the system and plan the capacity/resources accordingly to the expected load.

# Who should use this

- **Web Developers** who are building a backend/frontend solutions on top of the LLM service.
- **DevOps** who are responsible for planning the capacity/resources for the LLM service.
- **QAs** who are responsible for testing the LLM service.
- Anyone who is interested in the performance of the LLM service from availability and scalability perspective.

# Disclaimer

It is **important** to note that the tests conducted are not typical performance tests used to evaluate the behavior of
models. Instead, they focus on the service's ability to handle concurrent requests.

## Tooling

The load testing was performed using the following tools:

- **Apache Bench**: A benchmarking tool for web servers, used to simulate multiple concurrent requests to the LLM
  service.
- **Ollama**: A tool that runs large language models as HTTP services, facilitating the testing process.

### Install Apache Bench.

Windows: download binaries from https://apachelounge.com/download/
Mac: `brew install ab`

## Choosing the prompt

The goal of the `Imagine a joke ...` prompt is to ensure that the LLM service will provide non-repetitive and diverse
responses.
High temperature will force LLM to generate unique content even when prompted with similar or identical input.
By implementing these strategies, we can test the LLM service's performance under conditions that mimic real-world
usage, where users expect creative and non-repetitive responses.

## Models under test

```bash
ollama list
```

NAME ID SIZE MODIFIED           
llama3:latest 71a106a91016 4.7 GB 22 hours ago      	
llama2:latest 78e26419b446 3.8 GB About a minute ago
mistral:latest 61e88e884507 4.1 GB 1 second ago
dolphin-mistral:latest 5dc8c5a2be65 4.1 GB 2 minutes ago

## Test Environment

The tests were conducted on a local machine with the following specifications:

- MacBook Pro (16GB, 2021) Processor: Apple M1 Pro

## Run

```bash
verbosity=4
requests=100
concurrency=1
models=("llama3" "llama2" "mistral" "dolphin-mistral")

for model in "${models[@]}"; do
    echo "Testing Model: $model"
    prompt=$(cat prompt-template.json | sed "s/<MODEL_NAME>/$model/g")
    echo $prompt > prompt.json
    ab -v $verbosity -n $requests -c $concurrency -p prompt.json http://127.0.0.1:11434/api/generate > $model-results.txt
done
```

## Results

Detailed output of the test run can be found in the [results](results) directory.

# Results overview

Table of results:

| Model             | Total Time (min)  | Mean time per request (s) | Total transferred (KB) |
|-------------------|-------------------|---------------------------|------------------------|
| llama 3           | 6.25              | 3.8                       | 171                    |
| mistral           | 10.44             | 6.3                       | 241                    |
| dolphin-mistral   | 10.43             | 6.2                       | 252                    |
| llama 2           | 13.35             | 8.0                       | 306                    |

# Conclusion
llama3 model has the best performance in terms of requests per second and time per request, however the lower total transferred data might indicate that the model responses are shorter than the other models.



