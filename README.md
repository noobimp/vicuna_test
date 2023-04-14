# vicuna_test

直接启动推理的权重文件整理后可发出~后续微调的尝试也在github更新（如果有😊），感兴趣的老板可以点个star

新版本v1.1
我问它UCAS怎么样and写一道两数之和a+b=target，效果如下：

USER: How about University of Chinese Academy of Sciences

ASSISTANT: The University of Chinese Academy of Sciences (UCAS) is a research-oriented university located in Beijing, China. It is one of the leading universities in China and is highly regarded in the fields of natural sciences, technology, and engineering.中国科学院大学（UCAS）是一所位于中国北京的研究型大学。它是中国领先的大学之一，在自然科学、技术和工程领域享有很高的声誉。

UCAS has a highly diverse student population, with students from China and many other countries around the world. The university offers a wide range of programs in the natural sciences, engineering, and technology, as well as in the humanities and social sciences. It has a strong emphasis on research and offers opportunities for students to work on cutting-edge projects with leading researchers in their fields.UCAS 拥有高度多样化的学生群体，学生来自中国和世界上许多其他国家。该大学提供范围广泛的自然科学、工程和技术以及人文和社会科学课程。它非常重视研究，并为学生提供与各自领域的领先研究人员一起从事尖端项目的机会。

UCAS has a beautiful campus located in the heart of Beijing, with modern facilities and resources to support students' academic and personal development. The university has a strong international focus and offers a range of programs and initiatives to support international students, including language courses, cultural events, and exchange programs.国科大拥有位于北京市中心的美丽校园，拥有现代化的设施和资源，支持学生的学术和个人发展。该大学非常注重国际，并提供一系列计划和举措来支持国际学生，包括语言课程、文化活动和交流计划。

Overall, UCAS is a highly respected university with a strong reputation in the fields of natural sciences, technology, and engineering. It is a great choice for students who are interested in pursuing a career in these fields and who want to be part of a diverse and dynamic academic community.总体而言，国科大是一所备受推崇的大学，在自然科学、技术和工程领域享有盛誉。对于有兴趣在这些领域从事职业并希望成为多元化和充满活力的学术社区的一部分的学生来说，这是一个很好的选择。

USER: Given an array of integers nums and an integer target, return indices of the two numbers such that they add up to target.You may assume that each input would have exactly one solution, and you may not use the same element twice.You can return the answer in any order.
给定一个整数数组 nums 和一个整数 target，返回两个数字的索引，使它们加起来等于 target。您可以假设每个输入都有一个解决方案，并且您可能不会两次使用相同的元素.您可以按任何顺序返回答案。
ASSISTANT: Here is a Python function that solves the problem you described:这是一个 Python 函数，可以解决您描述的问题：
```
def twoSum(nums: List[int], target: int) -> List[int]:
    """
    Given an array of integers nums and an integer target, return the indices of the two numbers such that they add up to target.

    :param nums: A list of integers
    :param target: The target number to find the two numbers that add up to
    :return: A list of indices of the two numbers
    """
    nums.sort()
    for i in range(len(nums)):
        for j in range(i+1, len(nums)):
            if nums[i] + nums[j] == target:
                return [i, j]
    return []
```
This function first sorts the input list `nums` in ascending order. Then it iterates through the sorted list and checks if the difference between each number and the next higher number is equal to the target. If it is, it returns a list of indices of the two numbers. If not, it returns an empty list.此函数首先按升序对输入列表 `nums` 进行排序。然后它遍历排序列表并检查每个数字与下一个更高数字之间的差异是否等于目标。如果是，它返回两个数字的索引列表。如果不是，它返回一个空列表。

The function runs in O(n^2) time, where n is the length of the input list `nums`. This is because it sorts the list and then iterates through it twice, which takes O(n) time in the worst case. However, the number of iterations is usually very small (typically less than 10), so the overall running time is usually quite fast.该函数在 O(n^2) 时间内运行，其中 n 是输入列表的长度nums。这是因为它对列表进行排序，然后遍历它两次，这在最坏的情况下需要 O(n) 时间。但是，迭代次数通常很少（通常少于 10 次），因此整体运行时间通常很快。

USER: exit...

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

![](https://raw.githubusercontent.com/noobimp/vicuna_test/main/1.png#pic_center)


7B下载完成后是这5个文件，权重文件12.55G

### 3.权重转换
注意最新分支的转换脚本可能会报错，我用的是这个：
https://github.com/huggingface/transformers/blob/9eae4aa57650c1dbe1becd4e0979f6ad1e572ac0/src/transformers/models/llama/convert_llama_weights_to_hf.py

可以把这个脚本放在pyllama_data目录下，然后执行：
```
python convert_llama_weights_to_hf.py --input_dir ./ --model_size 7B --output_dir ./output/7B
```
得到output/7B

![](https://raw.githubusercontent.com/noobimp/vicuna_test/main/2.png#pic_center)


下面的命令可以自动下载vicuna的delta权重并和转换后的LLaMA权重融合，输出至target目录下
```
#base和target路径可以按你本地调整
python -m fastchat.model.apply_delta \
    --base ./pyllama_data/output/7B \
    --target ./vicuna_data/vicuna-7b  \
    --delta lmsys/vicuna-7b-delta-v1.1
```

此转换命令需要大约30GB内存，得到vicuna_data/vicuna-7b，总大小约14GB。

![](https://raw.githubusercontent.com/noobimp/vicuna_test/main/3.png#pic_center)
    
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
    
![](https://raw.githubusercontent.com/noobimp/vicuna_test/main/4.png#pic_center)

对于7B版本，让它讲讲UCAS怎么样，做一道两数之和a+b=target，大致效果如图。
