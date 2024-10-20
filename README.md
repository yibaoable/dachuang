# 处理git diff命令获得的diff_out内容并统计相关数据
## 概述

- 统计diffout中涉及的修改块数hunk， 所有修改块所在函数func的集合，涉及的java文件数 file。脚本`data_processing.py`
- 判断commit中的Java文件是否存在对应的测试文件。脚本`data_processing_testcase.py`
	- 输出格式`{file1:1,file2:2,file3:0}`  
		- 1：修改文件列表中有该文件对应的test文件
		- 2：修改文件列表中没有该文件对应的test文件，但是仓库中有
		- 0：不存在该文件对应的修改文件
	- 代码思路：
		- 对于每个仓库， 从diffout中获取修改文件列表
		1. 读取修改文件列表，如果是test文件，说明其对应的修改文件找到test，testcase=1
		2. 对于列表中没有对应test的java文件：
			- 获得仓库中所有测试用例和焦点方法的mapping，提取出焦点方法得到一个列表methodlist。（调用脚本find_map_test_case)
			- 获取该文件中所有焦点方法（tree-sitter），在methodlist中寻找是否存在该方法。存在，testcase=2.
		3. 以上两种方法都没找到，testcase=0


## 文件解释

- `data_processing_testcase.py` 扫描本地仓库，从已有的diff.txt文件中获取diffout内容，并判断每个修改文件是否有对应的测试文件。
- `find_map_test_case.py` 用于在某仓库中获得所有方法和对应测试用例的映射，提取map中的焦点方法形成一个列表，并输出到该仓库下的一个json文件里。由data_processing_testcase.py调用。
- `TestParser.py` 由find_map_test_case.py调用。
- `data_processing.py` 统计file/hunk/func等数据
- build文件夹：放置tree-sitter Java 语法文件

## 运行准备

- 安装tree-sitter，git clone java语言的仓库tree-sitter-java
```
	pip3 install tree_sitter
	git clone https://github.com/tree-sitter/tree-sitter-java

```
- 生成.so文件，运行下面的代码
```python
import tree_sitter
from tree_sitter import Language

Language.build_library(
  # so文件保存位置
  'build/my-languages.so',

  # git clone的tree-sitter-java仓库路径
  [
    'tree-sitter-java'
  ]
)
```

-  需要已经克隆到本地的仓库，每个仓库下有一个diff.txt文件。运行data_processing.py会进行克隆并输出diff.txt文件。
## 一些路径说明

- base_path1 = 'E:/dachuang/github_clone'  # 存放所有仓库的目录
- output_file = "E:/dachuang/output.csv"  # 输出文件
- input_csv = "E:/dachuang/output.csv"  # 输入文件，只需更新原来的输出文件的testcase列
-  output_dir = 'E:/dachuang/tmp/output/'  # 输出每个仓库文件夹的路径
- grammar_path = 'E:/dachuang/build/my-languages.so'  # tree-sitter Java 语法文件的路径
- git_bash_path = "E:/Git/git-bash.exe" # Git Bash 的路径

