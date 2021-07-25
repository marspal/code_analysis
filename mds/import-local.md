# import-local

``简介:`` 全局安装的包使用本地可用的安装版本

eg: 

npm i -g lerna 安装了全局lerna
mkdir project && cd project && npm i lerna -D 安装了一个本地的lerna

通过import-local 有多个版本直接使用当前目录安装的包, 是一个有用Cli工具包

核心逻辑: 
filename所在的目录 -> 找到pkg所在的目录 -> 计算filename和pkg所在的目录的相对路径
-> 读取pkg中的name -> 拼装name和相对路径 -> 通过process.cwd() + 拼装的路径 -> 找到对应的localfile
-> require localfile 加载运行文件

```js
module.exports = filename => {
	const globalDir = pkgDir.sync(path.dirname(filename));
	const relativePath = path.relative(globalDir, filename);
	const pkg = require(path.join(globalDir, 'package.json'));
	const localFile = resolveCwd.silent(path.join(pkg.name, relativePath));

	// Use `path.relative()` to detect local package installation,
	// because __filename's case is inconsistent on Windows
	// Can use `===` when targeting Node.js 8
	// See https://github.com/nodejs/node/issues/6624
	return localFile && path.relative(localFile, filename) !== '' ? require(localFile) : null;
};
```

知识点:
path.dirname(path)  返回path的目录名, 忽略Trailing目录
path.relative(from, to) 返回相对path从from到to基于当前工作目录; 相当于从from目录怎么到to目录
path.join([...paths]) 拼接多个路径

__filename: 当前文件所在目录