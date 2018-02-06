使用Metasploit将文件压缩成zip格式非常简单。对于大多数用途，您可以使用 Msf::Util::EXE.to_zip 来将数据压缩到一个zip文件

## 用法

```ruby
files =
  [
    {data: 'AAAA', fname: 'test1.txt', comment: 'my comment'},
    {data: 'BBBB', fname: 'test2.txt'}
  ]

zip = Msf::Util::EXE.to_zip(files)
```

如果保存为文件，则上面的示例将解压缩出以下内容：

```
$ unzip test.zip 
Archive:  test.zip
 extracting: test1.txt               
 extracting: test2.txt
```