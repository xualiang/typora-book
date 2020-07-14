## 下载并安装StarUML
http://staruml.io
注意：安装完成后运行一次软件，否则破解后会报“软件已被破坏”的错误。

## 破解
1) 安装npm
2) 安装asar

npm install asar -g

3) 进入目录，解压文件app.asar

cd /Applications/StarUML.app/Contents/Resources/
asar extract app.asar app

4) 修改新生成的app目录下的lisence文件

vim app/src/engine/license-manager.js 

5) 找到checkLicenseValidity()函数，125行开始的，原代码：

checkLicenseValidity () {
    this.validate().then(() => {
        setStatus(this, true)
    }, () => {
        setStatus(this, false)
        UnregisteredDialog.showDialog()
    })
}

修改为：

checkLicenseValidity () {
    this.validate().then(() => {
        setStatus(this, true)
    }, () => {
        setStatus(this, true)
    })
}

6) 打包覆盖原app.asar

asar pack app app.asar