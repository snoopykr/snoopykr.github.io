---
layout: single
title: "Google Colab에서 Fine-tuning"
categories : LLM
tag: [unsloth, colab, Fine-tuning]
---

Google Colab에서 무료로 Fine-tuning을 할수 있는 방법

### 설명
- unsloth가 제공하는 Google Colab에서 Fine-tuning을 하는 방법을 제시한 소스
- Fine-tuning에 대한 지식을 습득하기 좋은 샘플

To run this, press "*Runtime*" and press "*Run all*" on a **free** Tesla T4 Google Colab instance!
<div class="align-center">
  <a href="https://github.com/unslothai/unsloth"><img src="https://github.com/unslothai/unsloth/raw/main/images/unsloth%20new%20logo.png" width="115"></a>
  <a href="https://discord.gg/u54VK8m8tk"><img src="https://github.com/unslothai/unsloth/raw/main/images/Discord button.png" width="145"></a>
  <a href="https://ko-fi.com/unsloth"><img src="https://github.com/unslothai/unsloth/raw/main/images/Kofi button.png" width="145"></a></a> Join Discord if you need help + support us if you can!
</div>

To install Unsloth on your own computer, follow the installation instructions on our Github page [here](https://github.com/unslothai/unsloth#installation-instructions---conda).

You will learn how to do [data prep](#Data), how to [train](#Train), how to [run the model](#Inference), & [how to save it](#Save) (eg for Llama.cpp).

See on our [blog post](https://unsloth.ai/blog/gemma) on how we made **Gemma 7b 2.5x faster** and **Gemma 2b 2x faster**!


```python
%%capture
# Installs Unsloth, Xformers (Flash Attention) and all other packages!
!pip install "unsloth[colab-new] @ git+https://github.com/unslothai/unsloth.git"
!pip install --no-deps xformers "trl<0.9.0" peft accelerate bitsandbytes
```

* We support Llama, Mistral, CodeLlama, TinyLlama, Vicuna, Open Hermes etc
* And Yi, Qwen ([llamafied](https://huggingface.co/models?sort=trending&search=qwen+llama)), Deepseek, all Llama, Mistral derived archs.
* We support 16bit LoRA or 4bit QLoRA. Both 2x faster.
* `max_seq_length` can be set to anything, since we do automatic RoPE Scaling via [kaiokendev's](https://kaiokendev.github.io/til) method.
* [**NEW**] With [PR 26037](https://github.com/huggingface/transformers/pull/26037), we support downloading 4bit models **4x faster**! [Our repo](https://huggingface.co/unsloth) has Llama, Mistral 4bit models.


```python
from unsloth import FastLanguageModel
import torch
max_seq_length = 2048 # Choose any! We auto support RoPE Scaling internally!
dtype = None # None for auto detection. Float16 for Tesla T4, V100, Bfloat16 for Ampere+
load_in_4bit = True # Use 4bit quantization to reduce memory usage. Can be False.

# 4bit pre quantized models we support for 4x faster downloading + no OOMs.
fourbit_models = [
    "unsloth/mistral-7b-bnb-4bit",
    "unsloth/mistral-7b-instruct-v0.2-bnb-4bit",
    "unsloth/llama-2-7b-bnb-4bit",
    "unsloth/gemma-7b-bnb-4bit",
    "unsloth/gemma-7b-it-bnb-4bit", # Instruct version of Gemma 7b
    "unsloth/gemma-2b-bnb-4bit",
    "unsloth/gemma-2b-it-bnb-4bit", # Instruct version of Gemma 2b
] # More models at https://huggingface.co/unsloth

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/gemma-7b-bnb-4bit", # Choose ANY! eg teknium/OpenHermes-2.5-Mistral-7B
    max_seq_length = max_seq_length,
    dtype = dtype,
    load_in_4bit = load_in_4bit,
    # token = "hf_...", # use one if using gated models like meta-llama/Llama-2-7b-hf
)
```


    config.json:   0%|          | 0.00/1.11k [00:00<?, ?B/s]


    ==((====))==  Unsloth: Fast Gemma patching release 2024.3
       \\   /|    GPU: Tesla T4. Max memory: 14.748 GB. Platform = Linux.
    O^O/ \_/ \    Pytorch: 2.2.1+cu121. CUDA = 7.5. CUDA Toolkit = 12.1.
    \        /    Bfloat16 = FALSE. Xformers = 0.0.24. FA = False.
     "-____-"     Free Apache license: http://github.com/unslothai/unsloth


    /usr/local/lib/python3.10/dist-packages/transformers/quantizers/auto.py:155: UserWarning: You passed `quantization_config` or equivalent parameters to `from_pretrained` but the model you're loading already has a `quantization_config` attribute. The `quantization_config` from the model will be used.
      warnings.warn(warning_msg)



    model.safetensors:   0%|          | 0.00/5.57G [00:00<?, ?B/s]



    generation_config.json:   0%|          | 0.00/137 [00:00<?, ?B/s]



    tokenizer_config.json:   0%|          | 0.00/2.16k [00:00<?, ?B/s]



    tokenizer.model:   0%|          | 0.00/4.24M [00:00<?, ?B/s]



    tokenizer.json:   0%|          | 0.00/17.5M [00:00<?, ?B/s]



    special_tokens_map.json:   0%|          | 0.00/636 [00:00<?, ?B/s]


We now add LoRA adapters so we only need to update 1 to 10% of all parameters!


```python
model = FastLanguageModel.get_peft_model(
    model,
    r = 16, # Choose any number > 0 ! Suggested 8, 16, 32, 64, 128
    target_modules = ["q_proj", "k_proj", "v_proj", "o_proj",
                      "gate_proj", "up_proj", "down_proj",],
    lora_alpha = 16,
    lora_dropout = 0, # Supports any, but = 0 is optimized
    bias = "none",    # Supports any, but = "none" is optimized
    # [NEW] "unsloth" uses 30% less VRAM, fits 2x larger batch sizes!
    use_gradient_checkpointing = "unsloth", # True or "unsloth" for very long context
    random_state = 3407,
    use_rslora = False,  # We support rank stabilized LoRA
    loftq_config = None, # And LoftQ
)
```

    Unsloth 2024.3 patched 28 layers with 28 QKV layers, 28 O layers and 28 MLP layers.


<a name="Data"></a>
### Data Prep
We now use the Alpaca dataset from [yahma](https://huggingface.co/datasets/yahma/alpaca-cleaned), which is a filtered version of 52K of the original [Alpaca dataset](https://crfm.stanford.edu/2023/03/13/alpaca.html). You can replace this code section with your own data prep.

**[NOTE]** To train only on completions (ignoring the user's input) read TRL's docs [here](https://huggingface.co/docs/trl/sft_trainer#train-on-completions-only).

**[NOTE]** Remember to add the **EOS_TOKEN** to the tokenized output!! Otherwise you'll get infinite generations!

If you want to use the `ChatML` template for ShareGPT datasets, try our conversational [notebook](https://colab.research.google.com/drive/1Aau3lgPzeZKQ-98h69CCu1UJcvIBLmy2?usp=sharing).

For text completions like novel writing, try this [notebook](https://colab.research.google.com/drive/1ef-tab5bhkvWmBOObepl1WgJvfvSzn5Q?usp=sharing).


```python
alpaca_prompt = """Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.

### Instruction:
{}

### Input:
{}

### Response:
{}"""

EOS_TOKEN = tokenizer.eos_token # Must add EOS_TOKEN
def formatting_prompts_func(examples):
    instructions = examples["instruction"]
    inputs       = examples["input"]
    outputs      = examples["output"]
    texts = []
    for instruction, input, output in zip(instructions, inputs, outputs):
        # Must add EOS_TOKEN, otherwise your generation will go on forever!
        text = alpaca_prompt.format(instruction, input, output) + EOS_TOKEN
        texts.append(text)
    return { "text" : texts, }
pass

from datasets import load_dataset
dataset = load_dataset("yahma/alpaca-cleaned", split = "train")
dataset = dataset.map(formatting_prompts_func, batched = True,)
```


    Downloading readme:   0%|          | 0.00/11.6k [00:00<?, ?B/s]



    Downloading data:   0%|          | 0.00/44.3M [00:00<?, ?B/s]



    Generating train split: 0 examples [00:00, ? examples/s]



    Map:   0%|          | 0/51760 [00:00<?, ? examples/s]


<a name="Train"></a>
### Train the model
Now let's use Huggingface TRL's `SFTTrainer`! More docs here: [TRL SFT docs](https://huggingface.co/docs/trl/sft_trainer). We do 60 steps to speed things up, but you can set `num_train_epochs=1` for a full run, and turn off `max_steps=None`. We also support TRL's `DPOTrainer`!


```python
from trl import SFTTrainer
from transformers import TrainingArguments
from unsloth import is_bfloat16_supported

trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = max_seq_length,
    dataset_num_proc = 2,
    packing = False, # Can make training 5x faster for short sequences.
    args = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 4,
        warmup_steps = 5,
        max_steps = 60,
        learning_rate = 2e-4,
        fp16 = not is_bfloat16_supported(),
        bf16 = is_bfloat16_supported(),
        logging_steps = 1,
        optim = "adamw_8bit",
        weight_decay = 0.01,
        lr_scheduler_type = "linear",
        seed = 3407,
        output_dir = "outputs",
    ),
)
```


    Map (num_proc=2):   0%|          | 0/51760 [00:00<?, ? examples/s]


    /usr/local/lib/python3.10/dist-packages/accelerate/accelerator.py:432: FutureWarning: Passing the following arguments to `Accelerator` is deprecated and will be removed in version 1.0 of Accelerate: dict_keys(['dispatch_batches', 'split_batches', 'even_batches', 'use_seedable_sampler']). Please pass an `accelerate.DataLoaderConfiguration` instead: 
    dataloader_config = DataLoaderConfiguration(dispatch_batches=None, split_batches=False, even_batches=True, use_seedable_sampler=True)
      warnings.warn(



```python
#@title Show current memory stats
gpu_stats = torch.cuda.get_device_properties(0)
start_gpu_memory = round(torch.cuda.max_memory_reserved() / 1024 / 1024 / 1024, 3)
max_memory = round(gpu_stats.total_memory / 1024 / 1024 / 1024, 3)
print(f"GPU = {gpu_stats.name}. Max memory = {max_memory} GB.")
print(f"{start_gpu_memory} GB of memory reserved.")
```

    GPU = Tesla T4. Max memory = 14.748 GB.
    5.938 GB of memory reserved.



```python
trainer_stats = trainer.train()
```

    ==((====))==  Unsloth - 2x faster free finetuning | Num GPUs = 1
       \\   /|    Num examples = 51,760 | Num Epochs = 1
    O^O/ \_/ \    Batch size per device = 2 | Gradient Accumulation steps = 4
    \        /    Total batch size = 8 | Total steps = 60
     "-____-"     Number of trainable parameters = 50,003,968




    <div>

      <progress value='60' max='60' style='width:300px; height:20px; vertical-align: middle;'></progress>
      [60/60 08:54, Epoch 0/1]
    </div>
    <table border="1" class="dataframe">
  <thead>
 <tr style="text-align: left;">
      <th>Step</th>
      <th>Training Loss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>1.751900</td>
    </tr>
    <tr>
      <td>2</td>
      <td>2.306900</td>
    </tr>
    <tr>
      <td>3</td>
      <td>1.609100</td>
    </tr>
    <tr>
      <td>4</td>
      <td>1.755200</td>
    </tr>
    <tr>
      <td>5</td>
      <td>1.426000</td>
    </tr>
    <tr>
      <td>6</td>
      <td>1.370200</td>
    </tr>
    <tr>
      <td>7</td>
      <td>0.986500</td>
    </tr>
    <tr>
      <td>8</td>
      <td>1.162100</td>
    </tr>
    <tr>
      <td>9</td>
      <td>1.005800</td>
    </tr>
    <tr>
      <td>10</td>
      <td>1.075900</td>
    </tr>
    <tr>
      <td>11</td>
      <td>0.902600</td>
    </tr>
    <tr>
      <td>12</td>
      <td>0.938500</td>
    </tr>
    <tr>
      <td>13</td>
      <td>0.865400</td>
    </tr>
    <tr>
      <td>14</td>
      <td>1.012700</td>
    </tr>
    <tr>
      <td>15</td>
      <td>0.847900</td>
    </tr>
    <tr>
      <td>16</td>
      <td>0.853400</td>
    </tr>
    <tr>
      <td>17</td>
      <td>0.969700</td>
    </tr>
    <tr>
      <td>18</td>
      <td>1.194500</td>
    </tr>
    <tr>
      <td>19</td>
      <td>0.954200</td>
    </tr>
    <tr>
      <td>20</td>
      <td>0.843800</td>
    </tr>
    <tr>
      <td>21</td>
      <td>0.842700</td>
    </tr>
    <tr>
      <td>22</td>
      <td>0.877500</td>
    </tr>
    <tr>
      <td>23</td>
      <td>0.829100</td>
    </tr>
    <tr>
      <td>24</td>
      <td>0.937000</td>
    </tr>
    <tr>
      <td>25</td>
      <td>1.034500</td>
    </tr>
    <tr>
      <td>26</td>
      <td>1.017000</td>
    </tr>
    <tr>
      <td>27</td>
      <td>1.001900</td>
    </tr>
    <tr>
      <td>28</td>
      <td>0.847000</td>
    </tr>
    <tr>
      <td>29</td>
      <td>0.808900</td>
    </tr>
    <tr>
      <td>30</td>
      <td>0.830700</td>
    </tr>
    <tr>
      <td>31</td>
      <td>0.815100</td>
    </tr>
    <tr>
      <td>32</td>
      <td>0.846000</td>
    </tr>
    <tr>
      <td>33</td>
      <td>0.935700</td>
    </tr>
    <tr>
      <td>34</td>
      <td>0.799200</td>
    </tr>
    <tr>
      <td>35</td>
      <td>0.878700</td>
    </tr>
    <tr>
      <td>36</td>
      <td>0.814500</td>
    </tr>
    <tr>
      <td>37</td>
      <td>0.821100</td>
    </tr>
    <tr>
      <td>38</td>
      <td>0.726300</td>
    </tr>
    <tr>
      <td>39</td>
      <td>1.035300</td>
    </tr>
    <tr>
      <td>40</td>
      <td>1.112300</td>
    </tr>
    <tr>
      <td>41</td>
      <td>0.881100</td>
    </tr>
    <tr>
      <td>42</td>
      <td>0.901100</td>
    </tr>
    <tr>
      <td>43</td>
      <td>0.895500</td>
    </tr>
    <tr>
      <td>44</td>
      <td>0.845300</td>
    </tr>
    <tr>
      <td>45</td>
      <td>0.885600</td>
    </tr>
    <tr>
      <td>46</td>
      <td>0.885400</td>
    </tr>
    <tr>
      <td>47</td>
      <td>0.813100</td>
    </tr>
    <tr>
      <td>48</td>
      <td>1.125500</td>
    </tr>
    <tr>
      <td>49</td>
      <td>0.858400</td>
    </tr>
    <tr>
      <td>50</td>
      <td>1.005600</td>
    </tr>
    <tr>
      <td>51</td>
      <td>0.960500</td>
    </tr>
    <tr>
      <td>52</td>
      <td>0.895900</td>
    </tr>
    <tr>
      <td>53</td>
      <td>0.944900</td>
    </tr>
    <tr>
      <td>54</td>
      <td>1.107000</td>
    </tr>
    <tr>
      <td>55</td>
      <td>0.746100</td>
    </tr>
    <tr>
      <td>56</td>
      <td>1.003100</td>
    </tr>
    <tr>
      <td>57</td>
      <td>0.863500</td>
    </tr>
    <tr>
      <td>58</td>
      <td>0.796100</td>
    </tr>
    <tr>
      <td>59</td>
      <td>0.788700</td>
    </tr>
    <tr>
      <td>60</td>
      <td>0.867200</td>
    </tr>
  </tbody>
</table><p>



```python
#@title Show final memory and time stats
used_memory = round(torch.cuda.max_memory_reserved() / 1024 / 1024 / 1024, 3)
used_memory_for_lora = round(used_memory - start_gpu_memory, 3)
used_percentage = round(used_memory         /max_memory*100, 3)
lora_percentage = round(used_memory_for_lora/max_memory*100, 3)
print(f"{trainer_stats.metrics['train_runtime']} seconds used for training.")
print(f"{round(trainer_stats.metrics['train_runtime']/60, 2)} minutes used for training.")
print(f"Peak reserved memory = {used_memory} GB.")
print(f"Peak reserved memory for training = {used_memory_for_lora} GB.")
print(f"Peak reserved memory % of max memory = {used_percentage} %.")
print(f"Peak reserved memory for training % of max memory = {lora_percentage} %.")
```

    549.6228 seconds used for training.
    9.16 minutes used for training.
    Peak reserved memory = 11.002 GB.
    Peak reserved memory for training = 5.064 GB.
    Peak reserved memory % of max memory = 74.6 %.
    Peak reserved memory for training % of max memory = 34.337 %.


<a name="Inference"></a>
### Inference
Let's run the model! You can change the instruction and input - leave the output blank!


```python
# alpaca_prompt = Copied from above
FastLanguageModel.for_inference(model) # Enable native 2x faster inference
inputs = tokenizer(
[
    alpaca_prompt.format(
        "Continue the fibonnaci sequence.", # instruction
        "1, 1, 2, 3, 5, 8", # input
        "", # output - leave this blank for generation!
    )
], return_tensors = "pt").to("cuda")

outputs = model.generate(**inputs, max_new_tokens = 64, use_cache = True)
tokenizer.batch_decode(outputs)
```




    ['<bos>Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.\n\n### Instruction:\nContinue the fibonnaci sequence.\n\n### Input:\n1, 1, 2, 3, 5, 8\n\n### Response:\n13, 21, 34, 55, 89, 144<eos>']



 You can also use a `TextStreamer` for continuous inference - so you can see the generation token by token, instead of waiting the whole time!


```python
# alpaca_prompt = Copied from above
FastLanguageModel.for_inference(model) # Enable native 2x faster inference
inputs = tokenizer(
[
    alpaca_prompt.format(
        "Continue the fibonnaci sequence.", # instruction
        "1, 1, 2, 3, 5, 8", # input
        "", # output - leave this blank for generation!
    )
], return_tensors = "pt").to("cuda")

from transformers import TextStreamer
text_streamer = TextStreamer(tokenizer)
_ = model.generate(**inputs, streamer = text_streamer, max_new_tokens = 128)
```

    <bos>Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.
    
    ### Instruction:
    Continue the fibonnaci sequence.
    
    ### Input:
    1, 1, 2, 3, 5, 8
    
    ### Response:
    13, 21, 34, 55, 89, 144<eos>


<a name="Save"></a>
### Saving, loading finetuned models
To save the final model as LoRA adapters, either use Huggingface's `push_to_hub` for an online save or `save_pretrained` for a local save.

**[NOTE]** This ONLY saves the LoRA adapters, and not the full model. To save to 16bit or GGUF, scroll down!


```python
model.save_pretrained("lora_model") # Local saving
tokenizer.save_pretrained("lora_model")
# model.push_to_hub("your_name/lora_model", token = "...") # Online saving
# tokenizer.push_to_hub("your_name/lora_model", token = "...") # Online saving
```

Now if you want to load the LoRA adapters we just saved for inference, set `False` to `True`:


```python
if False:
    from unsloth import FastLanguageModel
    model, tokenizer = FastLanguageModel.from_pretrained(
        model_name = "lora_model", # YOUR MODEL YOU USED FOR TRAINING
        max_seq_length = max_seq_length,
        dtype = dtype,
        load_in_4bit = load_in_4bit,
    )
    FastLanguageModel.for_inference(model) # Enable native 2x faster inference

# alpaca_prompt = You MUST copy from above!

inputs = tokenizer(
[
    alpaca_prompt.format(
        "What is a famous tall tower in Paris?", # instruction
        "", # input
        "", # output - leave this blank for generation!
    )
], return_tensors = "pt").to("cuda")

outputs = model.generate(**inputs, max_new_tokens = 64, use_cache = True)
tokenizer.batch_decode(outputs)
```




    ['<bos>Below is an instruction that describes a task, paired with an input that provides further context. Write a response that appropriately completes the request.\n\n### Instruction:\nWhat is a famous tall tower in Paris?\n\n### Input:\n\n\n### Response:\nOne of the most famous tall towers in Paris is the Eiffel Tower. It is a wrought-iron lattice tower on the Champ de Mars in Paris, France. It is named after the engineer Gustave Eiffel, whose company designed and built the tower. The tower is 324 meters (1,063 feet']



You can also use Hugging Face's `AutoModelForPeftCausalLM`. Only use this if you do not have `unsloth` installed. It can be hopelessly slow, since `4bit` model downloading is not supported, and Unsloth's **inference is 2x faster**.


```python
if False:
    # I highly do NOT suggest - use Unsloth if possible
    from peft import AutoPeftModelForCausalLM
    from transformers import AutoTokenizer
    model = AutoPeftModelForCausalLM.from_pretrained(
        "lora_model", # YOUR MODEL YOU USED FOR TRAINING
        load_in_4bit = load_in_4bit,
    )
    tokenizer = AutoTokenizer.from_pretrained("lora_model")
```

### Saving to float16 for VLLM

We also support saving to `float16` directly. Select `merged_16bit` for float16 or `merged_4bit` for int4. We also allow `lora` adapters as a fallback. Use `push_to_hub_merged` to upload to your Hugging Face account! You can go to https://huggingface.co/settings/tokens for your personal tokens.


```python
# Merge to 16bit
if False: model.save_pretrained_merged("model", tokenizer, save_method = "merged_16bit",)
if False: model.push_to_hub_merged("hf/model", tokenizer, save_method = "merged_16bit", token = "")

# Merge to 4bit
if False: model.save_pretrained_merged("model", tokenizer, save_method = "merged_4bit",)
if False: model.push_to_hub_merged("hf/model", tokenizer, save_method = "merged_4bit", token = "")

# Just LoRA adapters
if False: model.save_pretrained_merged("model", tokenizer, save_method = "lora",)
if False: model.push_to_hub_merged("hf/model", tokenizer, save_method = "lora", token = "")
```

### GGUF / llama.cpp Conversion
To save to `GGUF` / `llama.cpp`, we support it natively now! We clone `llama.cpp` and we default save it to `q8_0`. We allow all methods like `q4_k_m`. Use `save_pretrained_gguf` for local saving and `push_to_hub_gguf` for uploading to HF.

Some supported quant methods (full list on our [Wiki page](https://github.com/unslothai/unsloth/wiki#gguf-quantization-options)):
* `q8_0` - Fast conversion. High resource use, but generally acceptable.
* `q4_k_m` - Recommended. Uses Q6_K for half of the attention.wv and feed_forward.w2 tensors, else Q4_K.
* `q5_k_m` - Recommended. Uses Q6_K for half of the attention.wv and feed_forward.w2 tensors, else Q5_K.


```python
# Save to 8bit Q8_0
if False: model.save_pretrained_gguf("model", tokenizer,)
if False: model.push_to_hub_gguf("hf/model", tokenizer, token = "")

# Save to 16bit GGUF
if False: model.save_pretrained_gguf("model", tokenizer, quantization_method = "f16")
if False: model.push_to_hub_gguf("hf/model", tokenizer, quantization_method = "f16", token = "")

# Save to q4_k_m GGUF
if False: model.save_pretrained_gguf("model", tokenizer, quantization_method = "q4_k_m")
if False: model.push_to_hub_gguf("hf/model", tokenizer, quantization_method = "q4_k_m", token = "")
```

Now, use the `model-unsloth.gguf` file or `model-unsloth-Q4_K_M.gguf` file in `llama.cpp` or a UI based system like `GPT4All`. You can install GPT4All by going [here](https://gpt4all.io/index.html).

And we're done! If you have any questions on Unsloth, we have a [Discord](https://discord.gg/u54VK8m8tk) channel! If you find any bugs or want to keep updated with the latest LLM stuff, or need help, join projects etc, feel free to join our Discord!

Some other links:
1. Zephyr DPO 2x faster [free Colab](https://colab.research.google.com/drive/15vttTpzzVXv_tJwEk-hIcQ0S9FcEWvwP?usp=sharing)
2. Llama 7b 2x faster [free Colab](https://colab.research.google.com/drive/1lBzz5KeZJKXjvivbYvmGarix9Ao6Wxe5?usp=sharing)
3. TinyLlama 4x faster full Alpaca 52K in 1 hour [free Colab](https://colab.research.google.com/drive/1AZghoNBQaMDgWJpi4RbffGM1h6raLUj9?usp=sharing)
4. CodeLlama 34b 2x faster [A100 on Colab](https://colab.research.google.com/drive/1y7A0AxE3y8gdj4AVkl2aZX47Xu3P1wJT?usp=sharing)
5. Mistral 7b [free Kaggle version](https://www.kaggle.com/code/danielhanchen/kaggle-mistral-7b-unsloth-notebook)
6. We also did a [blog](https://huggingface.co/blog/unsloth-trl) with 🤗 HuggingFace, and we're in the TRL [docs](https://huggingface.co/docs/trl/main/en/sft_trainer#accelerate-fine-tuning-2x-using-unsloth)!
7. `ChatML` for ShareGPT datasets, [conversational notebook](https://colab.research.google.com/drive/1Aau3lgPzeZKQ-98h69CCu1UJcvIBLmy2?usp=sharing)
8. Text completions like novel writing [notebook](https://colab.research.google.com/drive/1ef-tab5bhkvWmBOObepl1WgJvfvSzn5Q?usp=sharing)

<div class="align-center">
  <a href="https://github.com/unslothai/unsloth"><img src="https://github.com/unslothai/unsloth/raw/main/images/unsloth%20new%20logo.png" width="115"></a>
  <a href="https://discord.gg/u54VK8m8tk"><img src="https://github.com/unslothai/unsloth/raw/main/images/Discord.png" width="145"></a>
  <a href="https://ko-fi.com/unsloth"><img src="https://github.com/unslothai/unsloth/raw/main/images/Kofi button.png" width="145"></a></a> Support our work if you can! Thanks!
</div>
