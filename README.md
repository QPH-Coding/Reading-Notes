### 须知

---

- 如果想要完整的体验（如文件之间的链接），请使用`Obsidian`打开读书笔记。
- 每个文件夹是一本书名，同时也是`Obsidian`的一个库。
  
  
  
### 快速开始

---

- 可以在任意安装了git的shell内使用以下命令
  
  ```bash
  git clone https://github.com/QPH-Coding/Reading-Notes.git
  ```
- 如果是`Windows`系统，有可能会出现问题：
  - 如果提示`error: invalid path ...`：因为`Windows`会对`NTFS`格式的硬盘进行保护，可以在`git clone`前使用如下命令对git进行配置
    
    ```bash
    #单次配置
    git config core.protectNTFS false
    #全局配置
    git config --global core.protectNTFS false
    ```
