* 文件格式
    * pickle
        * pytorch使用
    * gguf(GPT-Generated Unified Format)
        * 参考: 
            * [一文搞懂大模型文件存储格式新宠GGUF](https://zhuanlan.zhihu.com/p/848013326)
            * [如何加载 GGUF 模型（分片/Shared/Split/00001-of-0000... GGUF 文件的加载解决方法）](https://zhuanlan.zhihu.com/p/853454088)
        * 由 Georgi Gerganov(llama.cpp的创始人)定义发布的一种大模型文件格式, 继承自其前身 GGML
        * 特性
            * 单文件部署: 它们可以轻松分发和加载, 并且不需要任何外部文件来获取附加信息. 
            * 可扩展性: 可以将新特征添加到基于 GGML 的执行器中/可以将新信息添加到 GGUF 模型中, 而不会破坏与现有模型的兼容性. 
            * mmap兼容性: 可以使用mmap加载模型, 以实现快速地加载和保存. 
            * 易于使用: 无论使用何种语言, 都可以使用少量代码轻松加载和保存模型, 无需外部库. 
            * 信息完整: 加载模型所需的所有信息都包含在模型文件中, 用户不需要提供任何额外的信息. 这大大简化了模型部署和共享的流程. 
        * 将gguf转回pytorch用的文件: 
            * [llama-cpp-torch](https://github.com/chu-tianxiang/llama-cpp-torch)
    * Safetensors:
        * 用于安全地存储张量的新格式, 是pickle格式的替代. 

# 环境搭建
* Windows
    * cuda
        * `nvidia-smi`查看最高支持的cuda版本
        * 下载: https://developer.nvidia.com/cuda-toolkit-archive
    * cudnn
        * 下载: https://developer.nvidia.com/cudnn-archive
        * 若下了tar包, 将其中`bin`, `include`, `lib`目录下的文件分别拷贝到CUDA目录下的各对应目录. 
        
# huggingface
* `pip install huggingface_hub`
* token: 命令行运行`huggingface-cli.exe login`, 按提示输入在huggingface网站上创建的token. 
* 下载模型
    * 方式一
        ```sh
            # zjunlp/knowlm-13b-ie 是模型名称
            huggingface-cli download --resume-download --local-dir-use-symlinks False zjunlp/knowlm-13b-ie --local-dir D:\Code\KnowLM\knowlm-13b-ie

            # 若需要token
            huggingface-cli download --resume-download zjunlp/knowlm-13b-ie --local-dir D:\Code\KnowLM\knowlm-13b-ie --local-dir-use-symlinks False --token hf_*****
        ```
```py
    from huggingface_hub import snapshot_download

    # 下载gemma-2b模型
    snapshot_download(
        repo_id="google/gemma-2b-flax", local_dir=r"D:\models\huggingface\hub\models--google--gemma-2b-flax\snapshots\4b7a4a7ffe3b60b6e6e995f0c925b84ce128072f"
    )

    # 下载的文件: 
        # 2b/              # Directory containing model weights
        # tokenizer.model  # Tokenizer
```
# ollama
* 基本命令
    * `pull <模型>`
    * `run <模型>`
    * `stop <模型>`
    * `rm <模型>`
    * `show <模型>`: 显示模型信息
    * `list`: 列出所有已下载的模型、
    * `ps`: 列出已加载到显存的模型. 
    * `--quantize`: 创建量化模型
        * 支持的量化: q4_0, q4_1, q5_0, q5_1, q8_0
        * K-means量化: q3_K_S, q3_K_M, q3_K_L, q4_K_S, q4_K_M, q5_K_S, q5_K_M, q6_K
* 模型地址: https://ollama.com/library
* 导入gguf文件
    * 创建`Modelfile`, 写入`FROM <gguf文件路径>`
    * 执行: `ollama create example -f Modelfile`
    * 执行: `ollama run example`
* 导入Safetensors文件
* 默认端口: 11434
* 环境变量
    * `OLLAMA_MODELS`: 模型文件的存放目录. 
    * `OLLAMA_HOST`: 默认`127.0.0.1`. 
    * `OLLAMA_PORT`: 默认11434. 
    * `OLLAMA_KEEP_ALIVE`: 大模型加载到内存的存活时间, 默认5分钟(5m). 
    * `OLLAMA_NUM_PARALLEL`: 并发请求处理的数量(默认1). 
    * `OLLAMA_MAX_QUEUE`: 请求队列长度(默认512). 
    * `OLLAMA_DEBUG`: 为1则输出调试日志. 
    * `OLLAMA_MAX_LOADED_MODELS`: 最多同时加载到内存中模型的数量(默认1). 
    * `OLLAMA_BASE_URL`: 协议, 主机名, 端口号 (默认`http://localhost:11434`)
    * `OLLAMA_API_KEY`: 形如`sk-11223344qwer`
* python
    * 参考
        * [ollama-python](https://github.com/ollama/ollama-python)
# JAX

# LangChain
* 参考
    * [LangChain与Ollama：重塑开发者的文本分析之旅](https://zhuanlan.zhihu.com/p/696293498)
    * [LangChain 中文入门教程](https://github.com/liaokongVFX/LangChain-Chinese-Getting-Started-Guide)


# OpenManus
* 参考
    * [通用型AI智能体Manus分析以及首个云平台自行搭建OpenManus](https://blog.csdn.net/lovely_yoshino/article/details/146094945)
    * [OpenManus 保姆级入门指南](https://blog.csdn.net/A79800/article/details/146177999)
* 要素:
    * 要求使用的模型支持`tools`

# tensorflow
* 注
    * Tensorflow 2.11(含)以后Windows版本不支持GPU
* 安装
    * windows
        * 参考: [在Win10/11 建立Tensorflow GPU訓練環境20230605](https://hackmd.io/@jerrychu/S1QvFG98h)
        * 环境: 
            * windows 11
            * python 3.9
            * cuda 11.8
            * cudnn 8.9.7
            * tensorflow 2.10: `pip install tensorflow==2.10`
        * 问题
            * `A module that was compiled using NumPy 1.x cannot be run in NumPy 2.0.1 as it may crash.`
                * numpy 2 的问题. 改装numpy 1: `pip install "numpy<2.0"`
* 使用
    ```py
        from tensorflow.python.client import device_lib
        print(device_lib.list_local_devices()) # 测试gpu的可用性

        import tensorflow as tf

        print("TensorFlow version:", tf.__version__) # tf的版本
        print("Is TensorFlow built with CUDA:", tf.test.is_built_with_cuda()) # 是否可用cuda
        print("Is GPU available:", tf.test.is_gpu_available()) # 是否可用gpu
    ```

# pytorch
* 安装
    * https://pytorch.org/get-started/locally/
    * 从上述链接中按提示选好环境, 复制所给pip命令, 安装pytorch
        * `cuda 12.1`对应的pytorch在安装时会带有`cudart64_12.dll`, `cudnn_cnn_train64_8.dll`等动态库(在`Lib\site-packages\torch\lib`下)

* 使用
    ```py
    
        import torch.utils.cpp_extension
        torch.utils.cpp_extension.CUDA_HOME # Pytorch 实际使用的运行时的 cuda 目录
        
        import torch
        torch.version.cuda # 编译该 Pytorch release 版本时使用的 cuda 版本

        torch.cuda.is_available() # 判断cuda是否可用
        torch.cuda.device_count() # 可用cuda的设备数量
    ```