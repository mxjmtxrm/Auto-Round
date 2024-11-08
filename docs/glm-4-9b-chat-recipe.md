## Model Details

This  is an int4 recipe with group_size 128 of  [THUDM/glm-4-9b-chat](https://huggingface.co/THUDM/glm-4-9b-chat) generated by [intel/auto-round](https://github.com/intel/auto-round). For GPTQ format, please load the model with revision `d45e33e`

## How To Use

### INT4 Inference on CPU/CUDA/HPU

```python
from auto_round import AutoRoundConfig ## must import for auto-round format
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

quantized_model_dir = "Intel/glm-4-9b-chat-int4-inc"

backend = "auto"  ##cuda, hpu, cpu(supported in auto_round>0.3.1),cuda:marlin(supported in auto_round>0.3.1 'pip install -v gptqmodel --no-build-isolation')
quantization_config = AutoRoundConfig(
    backend=backend
)
model = AutoModelForCausalLM.from_pretrained(quantized_model_dir,
                                             device_map=backend.split(':')[0],                                                        
                                             torch_dtype=torch.float16,
                                             quantization_config=quantization_config,
                                             trust_remote_code=True
                                            ).eval()

tokenizer = AutoTokenizer.from_pretrained(quantized_model_dir, trust_remote_code=True)
query = "请介绍一下智谱华章科技有限公司"
inputs = tokenizer.apply_chat_template([{"role": "user", "content": query}],
                                       add_generation_prompt=True,
                                       tokenize=True,
                                       return_tensors="pt",
                                       return_dict=True
                                       )
inputs = inputs.to(model.device)

gen_kwargs = {"max_length": 50, "do_sample": False, "top_k": 1}##change this to follow official usage
with torch.no_grad():
    outputs = model.generate(**inputs, **gen_kwargs)
    outputs = outputs[:, inputs['input_ids'].shape[1]:]
    print(tokenizer.decode(outputs[0], skip_special_tokens=True))

##请介绍一下智谱华章科技有限公司
#智谱华章科技有限公司是一家专注于人工智能、大数据、云计算等前沿技术领域的创新型企业。公司成立于2016年，总部位于中国北京，是一家集技术研发、产品开发、


##9.8大还是9.11大
##9.8比9.11小。在数值上，9.8小于9.11。


##Once upon a time,

#In a land where the sun kissed the horizon with a golden glow and the stars whispered secrets to the night, there was a village nestled among rolling hills and whispering forests. This was a place


##There is a girl who likes adventure,
##That's quite the intriguing starting point! If you're looking to create a story or a character, here's a brief introduction to a girl who likes adventure:

##---

##**Name:**
```



### Evaluate the model

pip3 install lm-eval==0.4.5

pip3 install langdetect,immutabledict,antlr4-python3-runtime==4.11

```bash
auto-round --eval --eval_bs 16 --tasks leaderboard_ifeval,leaderboard_mmlu_pro,gsm8k,lambada_openai,hellaswag,piqa,winogrande,truthfulqa_mc1,openbookqa,boolq,arc_easy,arc_challenge,cmmlu,ceval-valid

```

| Metric                                     | BF16   | INT4(6.4G) | INT4-quanted-lm-head(5.5G) |
| ------------------------------------------ | ------ | ---------- | -------------------------- |
| Avg                                        | 0.6260 | 0.6230     | 0.6204                     |
| leaderboard_mmlu_pro 5shot                 | 0.3678 | 0.3616     | 0.3610                     |
| leaderboard_ifeval inst_level_strict_acc   | 0.5504 | 0.5600     | 0.5588                     |
| leaderboard_ifeval prompt_level_strict_acc | 0.4067 | 0.4233     | 0.4067                     |
| cmmlu                                      | 0.7213 | 0.7137     | 0.7086                     |
| ceval-valid                                | 0.7065 | 0.7058     | 0.6909                     |
| lambada_openai                             | 0.6608 | 0.6493     | 0.6470                     |
| hellaswag                                  | 0.6195 | 0.6137     | 0.6134                     |
| winogrande                                 | 0.7561 | 0.7545     | 0.7522                     |
| piqa                                       | 0.8030 | 0.7976     | 0.8003                     |
| truthfulqa_mc1                             | 0.4223 | 0.4223     | 0.4284                     |
| openbookqa                                 | 0.3560 | 0.3640     | 0.3580                     |
| boolq                                      | 0.8691 | 0.8606     | 0.8578                     |
| arc_easy                                   | 0.8241 | 0.8249     | 0.8203                     |
| arc_challenge                              | 0.5469 | 0.5341     | 0.5444                     |
| gsm8k 5shot strict match                   | 0.7794 | 0.7597     | 0.7589                     |



### Generate the model

Here is the sample command to generate the model.  AutoRound should include this pr https://github.com/intel/auto-round/pull/304

```bash
auto-round \
--model THUDM/glm-4-9b-chat \
--iter 1000 \
--nsamples 512 \
--disable_eval \
--format "auto_round,auto_gptq" \
--model_dtype "fp16" \
--output_dir "./tmp_autoround"
```

copy all the *.py file to the quantized_model folder

For gptq format, need to add "block_name_to_quantize":"transformer.encoder.layers" to config.json, we only tested it on transformers==4.46.1

## Ethical Considerations and Limitations

The model can produce factually incorrect output, and should not be relied on to produce factually accurate information. Because of the limitations of the pretrained model and the finetuning datasets, it is possible that this model could generate lewd, biased or otherwise offensive outputs.

Therefore, before deploying any applications of the model, developers should perform safety testing.

## Caveats and Recommendations

Users (both direct and downstream) should be made aware of the risks, biases and limitations of the model.

Here are a couple of useful links to learn more about Intel's AI software:

* Intel Neural Compressor [link](https://github.com/intel/neural-compressor)
* Intel Extension for Transformers [link](https://github.com/intel/intel-extension-for-transformers)

## Disclaimer

The license on this model does not constitute legal advice. We are not responsible for the actions of third parties who use this model. Please consult an attorney before using this model for commercial purposes.



## Cite

@article{cheng2023optimize, title={Optimize weight rounding via signed gradient descent for the quantization of llms}, author={Cheng, Wenhua and Zhang, Weiwei and Shen, Haihao and Cai, Yiyang and He, Xin and Lv, Kaokao and Liu, Yi}, journal={arXiv preprint arXiv:2309.05516}, year={2023} }

[arxiv](https://arxiv.org/abs/2309.05516) [github](https://github.com/intel/auto-round)