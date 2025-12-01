---
{"dg-publish":true,"permalink":"/mac-init/"}
---



## brew

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Java

```
brew update
```

```
brew search adoptopenjdk
```

```
brew install --cask adoptopenjdk@11
```

## zsh

```
brew install zsh
```


## git

```
brew install git
```

安装完成后需要将本机的密钥注册到远端，否则无法clone代码