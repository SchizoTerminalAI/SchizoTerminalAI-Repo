<h1 align="center">GPT4All</h1>
<p align="center">Demo, data and code to train an assistant-style large language model with ~800k GPT-3.5-Turbo Generations based on LLaMa</p>

<p align="center">
<a href="https://s3.amazonaws.com/static.nomic.ai/gpt4all/2023_GPT4All_Technical_Report.pdf">:green_book: Technical Report</a>
</p>
<p align="center">
<a href="https://discord.gg/kvmy6dQB">Discord</a>
</p>



![gpt4all-lora-demo](https://user-images.githubusercontent.com/13879686/228352356-de66ca7a-df70-474e-b929-2e3656165051.gif)

Run on M1 Mac (not sped up!)

[Sample generations, see how well it performs.](./README-SAMPLES.md)

# Try it yourself

Here's how to get started with the CPU quantized gpt4all model checkpoint:

1. Download the `gpt4all-lora-quantized.bin` file from [Direct Link](https://the-eye.eu/public/AI/models/nomic-ai/gpt4all/gpt4all-lora-quantized.bin) or [[Torrent-Magnet]](https://tinyurl.com/gpt4all-lora-quantized).
2. Clone this repository, navigate to `chat`, and place the downloaded file there.
3. Run the appropriate command for your OS:
   - M1 Mac/OSX: `cd chat;./gpt4all-lora-quantized-OSX-m1`
   - Linux: `cd chat;./gpt4all-lora-quantized-linux-x86`
   - Windows (PowerShell): `cd chat;./gpt4all-lora-quantized-win64.exe`
   - Intel Mac/OSX: `cd chat;./gpt4all-lora-quantized-OSX-intel`

For custom hardware compilation, see our [Alpaca C++](https://github.com/zanussbaum/gpt4all.cpp) repository.

-----------

[Secret Unfiltered Checkpoint](https://the-eye.eu/public/AI/models/nomic-ai/gpt4all/gpt4all-lora-unfiltered-quantized.bin) - [[Torrent]](https://the-eye.eu/public/AI/models/nomic-ai/gpt4all/gpt4all-lora-unfiltered-quantized.bin.torrent)

This model had all refusal to answer responses removed from training. Try it with:
- `cd chat;./gpt4all-lora-quantized-OSX-m1 -m gpt4all-lora-unfiltered-quantized.bin`

-----------
Note: the full model on GPU (16GB of RAM required) performs much better in our qualitative evaluations.

# Python Client
## CPU Interface
To get running using the python client with the CPU interface, first install the [nomic client](https://github.com/nomic-ai/nomic) using `pip install nomic`
Then, you can use the following script to interact with GPU4All:
```
from nomic.gpt4all import GPT4All
m = GPT4All()
m.open()
m.prompt('write me a story about a lonely computer')
```

## GPU Interface
There are two ways to get up and running with this model on GPU.
The setup here is slightly more involved than the CPU model.
1. clone the nomic client [repo](https://github.com/nomic-ai/nomic) and run `pip install .[GPT4All]` in the home dir.
2. run `pip install nomic` and install the additional deps from the wheels built [here](https://github.com/nomic-ai/nomic/tree/main/bin)

Once this is done, you can run the model on GPU with a script like the following:
```
from nomic.gpt4all import GPT4AllGPU
m = GPT4AllGPU(LLAMA_PATH)
config = {'num_beams': 2,
          'min_new_tokens': 10,
          'max_length': 100,
          'repetition_penalty': 2.0}
out = m.generate('write me a story about a lonely computer', config)
print(out)
```
Where LLAMA_PATH is the path to a Huggingface Automodel compliant LLAMA model.
Nomic is unable to distribute this file at this time.
We are working on a GPT4All that does not have this limitation right now.

You can pass any of the [huggingface generation config params](https://huggingface.co/docs/transformers/main_classes/text_generation#transformers.GenerationConfig) in the config.

# Roadmap
## Short Term
 - <span style="color:green">(IN PROGRESS)</span> Train a GPT4All model based on GPTJ to alleviate llama distribution issues.
 - <span style="color:green">(IN PROGRESS)</span> Create improved CPU and GPU interfaces for this model.
 - <span style="color:red">(NOT STARTED)</span> Integrate llama.cpp bindings
 - <span style="color:red">(NOT STARTED)</span> Create a good conversational chat interface for the model.
 - <span style="color:red">(NOT STARTED)</span> Allow users to opt in and submit their chats for subsequent training runs

## Medium Term
 - <span style="color:red">(NOT STARTED)</span> Integrate GPT4All with [Atlas](https://atlas.nomic.ai) to allow for document retrieval.
   - BLOCKED by GPT4All based on GPTJ
 - <span style="color:red">(NOT STARTED)</span> Integrate GPT4All with Langchain.
 - <span style="color:green">(IN PROGRESS)</span> Build easy custom training scripts to allow users to fine tune models.

## Long Term
 - <span style="color:red">(NOT STARTED)</span> Allow anyone to curate training data for subsequent GPT4All releases using Atlas.
 - <span style="color:green">(IN PROGRESS)</span> Democratize AI. 

# Reproducibility

Trained LoRa Weights:
- gpt4all-lora (four full epochs of training):  https://huggingface.co/nomic-ai/gpt4all-lora
- gpt4all-lora-epoch-2 (three full epochs of training) https://huggingface.co/nomic-ai/gpt4all-lora-epoch-2

Raw Data:
- [Training Data Without P3](https://huggingface.co/datasets/nomic-ai/gpt4all_prompt_generations)
  - Explorer: https://atlas.nomic.ai/map/gpt4all_data_clean_without_p3
- [Full Dataset with P3](https://huggingface.co/datasets/nomic-ai/gpt4all_prompt_generations_with_p3)
  - Explorer: https://atlas.nomic.ai/map/gpt4all_data_clean

We are not distributing a LLaMa 7B checkpoint.

You can reproduce our trained model by doing the following:

## Setup

Clone the repo

`git clone --recurse-submodules https://github.com/nomic-ai/gpt4all.git`

`git submodule configure && git submodule update`

Setup the environment

```
python -m pip install -r requirements.txt

cd transformers
pip install -e . 

cd ../peft
pip install -e .
```

## Training

```bash
accelerate launch --dynamo_backend=inductor --num_processes=8 --num_machines=1 --machine_rank=0 --deepspeed_multinode_launcher standard --mixed_precision=bf16  --use_deepspeed --deepspeed_config_file=configs/deepspeed/ds_config.json train.py --config configs/train/finetune-7b.yaml
```

## Generate

```bash
python generate.py --config configs/generate/generate.yaml --prompt "Write a script to reverse a string in Python"
```
    
## Citation

If you utilize this reposistory, models or data in a downstream project, please consider citing it with:
```
@misc{gpt4all,
  author = {Yuvanesh Anand and Zach Nussbaum and Brandon Duderstadt and Benjamin Schmidt and Andriy Mulyar},
  title = {GPT4All: Training an Assistant-style Chatbot with Large Scale Data Distillation from GPT-3.5-Turbo},
  year = {2023},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{https://github.com/nomic-ai/gpt4all}},
}
```
