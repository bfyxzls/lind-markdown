# python笔记
## 依赖包

* 依赖包文件：requirements.txt 安装整个requirements.txt, pip --default-timeout=100 install --no-cache-dir -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/
* 安装某个本地包：pip install sdist/tools.tar.gz
* 使用阿里加速器 pip install flask -i https://mirrors.aliyun.com/pypi/simple/

## 全局镜像加速器

打开C:\Users\User\AppData\Roaming\pip\pip.ini

```cfg
[global]
timeout = 6000
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```

## pypi配置token
Using this token

To use this API token:

```
Set your username to __token__
Set your password to the token value, including the pypi- prefix
```

For example, if you are using Twine to upload your projects to PyPI, set up your `$USER_HOME/.pypirc` file like this:

```cfg
username = __token__
password = pypi-AgEIcHlwaS5vcmcCJGNkMWM5ODI3LWRjYzUtNGRmZi04MTRkLWZhOTFkNjczYjBmOAACKlszLCI4OWUzMmYwOC1kZmRhLTQyMjAtOTllZS05NzEyZDVmNjZlZWEiXQAABiDdBZu946onofiPTmjYzUAbS27VpWVSqyZ_fVJZcJrqKQ
```

从上面英文来翻译就是，在你的用户根目录，添加文件pypi，将下面内容放上去，注册username是你的真实名字，password是从pypi里申请的token

```cfg
[pypi]
username = bfyxzls
password = pypi-AgEIcHlwaS5vcmcCJGNkMWM5ODI3LWRjYzUtNGRmZi04MTRkLW
```
## 配置PyPI的账号密码信息

如果你没有配置 pip，可以使用以下命令来配置：

```shell
pip config set global.username <your-username>
pip config set global.password <your-password>
```

如果你使用了两步验证，那么你需要使用 API 令牌（API token）来进行身份验证。你可以通过访问 PyPI 的个人资料页面获取 API 令牌，然后将该令牌作为密码输入到 pip 中。

```shell
pip config set global.password <your-api-token>
```
