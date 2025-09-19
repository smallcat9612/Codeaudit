进入后台，在文件管理-上传设置-附件上传处可以添加可上传文件类型，
<img width="1683" height="662" alt="image" src="https://github.com/user-attachments/assets/57cf6146-1d37-4a37-9224-dddb66bd1036" />

按照正常逻辑来讲php后，是可以添加正常的php文件，但是在添加后会自动去除掉，
<img width="1412" height="296" alt="image" src="https://github.com/user-attachments/assets/091655c4-1f70-4040-b2b1-cd413145a841" />

修改后调用文件save.php ，从save文件代码段可以看到默认屏蔽php，用户在添加了php后会自动被替换为空。
具体包含以下：(array('php','bat','js','.',';'),'',$fileext)
<img width="1301" height="260" alt="image" src="https://github.com/user-attachments/assets/2c9aff57-0d6d-4b44-a59e-91a97b86a5c2" />

但是这里并未屏蔽大小写，shell无论linux还是windows是不受大小写影响的。
<img width="982" height="300" alt="image" src="https://github.com/user-attachments/assets/5952d32b-45ec-4828-9b50-20d6c3e2d8c1" />

在图片上传处也进行了添加Php，随后随便找一处进行上传测试。


<img width="1420" height="585" alt="image" src="https://github.com/user-attachments/assets/5bf7e2cd-a9a9-4bed-ba1e-6089dc07813a" />

<img width="679" height="259" alt="image" src="https://github.com/user-attachments/assets/146304e1-56f7-4336-817f-b4ed295e3630" />
