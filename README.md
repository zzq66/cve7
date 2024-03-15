NAME OF AFFECTED PRODUCT(S):lepton-cms 
AFFECTED AND/OR FIXED VERSION(S):7.1.0 
PROBLEM TYPE:File Upload; 
Impact:Arbitrary code execution; 
DESCRIPTION:An arbitrary file upload vulnerability in LeptonCMS v7.1.0 allows authenticated attackers to execute arbitrary code via uploading a PHP file.

![image](https://github.com/zzq66/cve7/assets/68541960/2a2fdf89-352a-4b54-998d-ef0ce834dc4c)# cve7
poc
复现流程：
登录后台转到settings修改Allowed filetypes on upload，增加一个php文件可上传类型。
![image](https://github.com/zzq66/cve7/assets/68541960/5cce957e-89a7-4a98-a60f-23ee61cc575b)

随后去media：
![image](https://github.com/zzq66/cve7/assets/68541960/c7c5f8cc-7768-45cd-a93c-033a997a59af)

直接上传php文件：
![image](https://github.com/zzq66/cve7/assets/68541960/ff506df6-2418-4ca5-aa87-3048399751ef)

![image](https://github.com/zzq66/cve7/assets/68541960/8a606e1e-f2c0-400a-81d1-13fc2ff31bc3)

上传成功：
![image](https://github.com/zzq66/cve7/assets/68541960/56336973-c97a-473d-a531-522f31324cbb)


点击show url：
![image](https://github.com/zzq66/cve7/assets/68541960/cbaba0a3-3739-4461-87c2-ff420800afc7)

访问发现：
![image](https://github.com/zzq66/cve7/assets/68541960/ddb41fa9-0e8a-40a2-9c8e-28b78e03fe74)




代码审计:
直接看到关键位置。我们在设置允许上传的文件类型的时候代码如下：
upload/backend/settings/save.php:
放入到了TABLE_PREFIX."settings:
![image](https://github.com/zzq66/cve7/assets/68541960/aeca732d-49af-49b9-bf9b-6b92f6c44edb)

在upload/modules/lib_r_filemanager/filemanager/config/config.php中代码如下，可以看到使用了TABLE_PREFIX."settings中的内容。

'ext'=> array_merge(
// $config['ext_img'],
// $config['ext_file'],
// $config['ext_misc'],
// $config['ext_video'],
// $config['ext_music'],
// allow only file-types from LEPTON -> options/upload_whitelist
explode(",",LEPTON_database::getInstance()->get_one("SELECT value FROM ".TABLE_PREFIX."settings WHERE name = 'upload_whitelist' "))
),

其中间没有任何过滤php后缀的行为，也就是说我们设置php后缀允许上传是可行的。
后续就是使用系统默认的上传文件逻辑，直接将文件上传。

缓解措施:设置不运行php相关危险后缀文件进行上传。


