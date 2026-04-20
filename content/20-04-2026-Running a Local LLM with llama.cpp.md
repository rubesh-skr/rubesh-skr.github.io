Title: Running a Local LLM with llama.cpp
Day 5 of the GenAI Learning Challenge focused on a major shift in how I interact with large language models. Instead of using cloud APIs, I set up and ran a real LLM locally on my machine. This elimin
Rubesh
Apr 20, 2026

Title: Running a Local LLM with llama.cpp (Day 5)
Date: 2026-04-20
Category: GenAI
Tags: GenAI, LLM, llama.cpp, GGUF, LocalAI
Slug: day5-llama-local-llm
Status: draft



Day 5 of the GenAI Learning Challenge focused on a major shift in how I interact with large language models. Instead of using cloud APIs, I set up and ran a real LLM locally on my machine. This eliminated the need for API keys, internet dependency, and external services. The entire process demonstrated that modern AI models can be executed directly on personal hardware using the right tools and optimizations.

Objective
The primary goal of this task was to understand how local inference works and to build a working pipeline using a lightweight model. The steps included installing the inference engine, downloading a quantized model, running it through a command-line interface, and integrating it into a Python script for programmatic usage.

Environment Setup
I used the prebuilt version of llama.cpp, which removes the need to compile from source. This made the setup faster and avoided platform-specific build issues.

The folder structure was organized as follows:

The executable file (llama-cli.exe) was placed inside the main directory

A models folder was created to store the model file

The GGUF model file was placed inside the models directory

This structure ensured that the command-line tool could correctly locate and load the model during execution.

Model Selection and Download
The model used in this task was TinyLlama in GGUF format, downloaded from Hugging Face. The specific version used was a quantized model (Q4_K_M), which balances performance and memory usage.

GGUF is a file format designed specifically for efficient local inference. It stores both model weights and metadata in a single file, making it easy to load and run without additional configuration. Quantization reduces the size of the model significantly, allowing it to run on CPU-based systems with limited memory.

Running the Model Using CLI
Once the setup was complete, I executed the model using the command-line interface provided by llama.cpp. The command used was:

.\llama-cli.exe -m models/tinyllama.gguf -p "What is Artificial Intelligence?"
In this command:

-m specifies the path to the model file

-p defines the input prompt

The model successfully generated a response directly in the terminal. This confirmed that the model was running locally without any external dependencies.

Python Integration
After validating the CLI execution, the next step was to integrate the model into a Python program using llama-cpp-python.

The Python script loads the model and sends a prompt programmatically:

from llama_cpp import Llama

llm = Llama(model_path="models/tinyllama.gguf")

response = llm(
    "Explain Artificial Intelligence in simple terms",
    max_tokens=100
)

print(response["choices"][0]["text"])
This step is important because it transforms the model from a command-line tool into a reusable component that can be integrated into applications such as chatbots, APIs, or automation workflows.

Challenges Faced
During the process, I encountered environment-related issues, particularly with Python package installation. The main issue was a mismatch between different Python environments (system Python vs Anaconda). The module was installed in one environment but executed in another, causing import errors.

This was resolved by standardizing the environment and ensuring that both installation and execution used the same Python interpreter. Using a consistent environment is critical when working with external libraries.

Another issue involved command syntax errors while running the CLI. These were resolved by carefully verifying command structure and argument spacing.

Key Learnings
Local inference allows running AI models without internet or API access

GGUF format is optimized for performance and simplicity in local setups

Quantized models enable efficient execution on CPU systems

Environment consistency is essential when working with Python dependencies

llama.cpp provides a lightweight and effective way to run LLMs locally

Importance of Local LLMs
Running models locally has several advantages:

Full control over data privacy

No dependency on third-party APIs

No usage-based cost

Ability to run offline applications

This approach is particularly useful for building private AI tools, experimenting with models, and understanding how inference works at a lower level



.

Conclusion
Day 5 marked a significant milestone in the learning journey. I successfully transitioned from using cloud-based AI services to running a fully local LLM. This experience provided a deeper understanding of model execution, optimization, and integration.

The ability to run an LLM locally opens up many possibilities for future projects, including building chat interfaces, integrating AI into applications, and experimenting with different models without external constraints.

The next step is to build on this foundation and create interactive applications using the local model.

