# vicuna_test
FastChat/vicuna项目连接 https://github.com/lm-sys/FastChat

新版本v1.1
### 1.安装fastchat和transformers
```
pip install fschat
# Install the latest main branch of huggingface/transformers
pip install git+https://github.com/huggingface/transformers
```
### 2.下载LLaMA权重
该项目也是由Meta的LLaMA 7B微调而来，因此需要下载LLaMA权重：可以从这里申请并下载LLaMA，也可以：
```
pip install pyllama -U
python -m llama.download --model_size 7B
#python -m llama.download --model_size 13B
```

![figure 1.](https://raw.githubusercontent.com/noobimp/vicuna_test/main/1.png#pic_center)


7B下载完成后是这5个文件，权重文件12.55G

### 3.权重转换
注意最新分支的转换脚本可能会报错，我用的是这个：
https://github.com/huggingface/transformers/blob/9eae4aa57650c1dbe1becd4e0979f6ad1e572ac0/src/transformers/models/llama/convert_llama_weights_to_hf.py

可以把这个脚本放在pyllama_data目录下，然后执行：
```
python convert_llama_weights_to_hf.py --input_dir ./ --model_size 7B --output_dir ./output/7B
```
得到output/7B

<div align=center>
  ![figure 2.](https://raw.githubusercontent.com/noobimp/vicuna_test/main/2.png)
</div>

下面的命令可以自动下载vicuna的delta权重并和转换后的LLaMA权重融合，输出至target目录下
```
#base和target路径可以按你本地调整
python -m fastchat.model.apply_delta \
    --base ./pyllama_data/output/7B \
    --target ./vicuna_data/vicuna-7b  \
    --delta lmsys/vicuna-7b-delta-v1.1
```

此转换命令需要大约30GB内存，得到vicuna_data/vicuna-7b，总大小约14GB。
<div align=center>
  ![figure 3.](https://raw.githubusercontent.com/noobimp/vicuna_test/main/3.png)
</div>
    
### 4.推理
GPU

启动推理Vicuna-13B大约需要28GB显存，Vicuna-7B需要14GB显存，可以通过--num-gpus多卡推理。

```
python -m fastchat.serve.cli --model-path ./vicuna_data/vicuna-7b --num-gpus 2
```

CPU

Vicuna-13B大约需要60GB内存，Vicuna-7B大约需要30GB内存。

```
python -m fastchat.serve.cli --model-path ./vicuna_data/vicuna-7b --device cpu
```

可使用--load-8bit压缩，“这可以将内存使用量减少大约一半，同时模型质量略有下降，8bit压缩的 Vicuna-13B可以在单个 NVIDIA 3090/4080/V100(16GB) GPU 上运行。”
而8bit压缩的7B在CPU上启动后，内存占用约5G，较短的回答内存占用约8-9G，速度感人，我估计10s才1-2个token。
    
<div align=center>
  ![figure 4.](https://raw.githubusercontent.com/noobimp/vicuna_test/main/4.png)
</div>

对于7B版本，让它讲讲UCAS怎么样，做一道两数之和a+b=target，大致效果如图。
可直接启动推理的权重文件可整理后发出~
