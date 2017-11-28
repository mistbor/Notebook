## How to use gitbook in Linux? 
**安装**

1. 首先安装nodejs
sudo apt-get install nodejs
sudo apt-get install npm
npm install gitbook-pdf -g

2. 由于生成pdf文件依赖于ebook-convert，故首先安装ebook-convert：http://calibre-ebook.com/download

3. 安装gitbook
npm install gitbook-cli -g

4. 安装pdf转换工具 https://calibre-ebook.com/download_linux


**使用**

1. 将项目clone到本地后，在项目根目录下执行：gitbook init, 则会生成相应的文件：SUMMARY.md 和 README.md 其中，SUMMARY.md是用来存放书的目录的，可以对SUMMARY.md进行编写，但是，对于目录的每一个链接必须有实体文件，否则点击无效。
2. 对md文件进行编辑保存后，使用gitbook pdf命令生成书即可。

