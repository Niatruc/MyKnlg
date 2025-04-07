* 文件格式
    * pickle
        * pytorch使用
    * gguf(GPT-Generated Unified Format)
        * 参考: [一文搞懂大模型文件存储格式新宠GGUF](https://zhuanlan.zhihu.com/p/848013326)
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
    * `OLLAMA_API_KEY`
* python
    * 
# JAX

# LangChain
* 参考
    * [LangChain与Ollama：重塑开发者的文本分析之旅](https://zhuanlan.zhihu.com/p/696293498)