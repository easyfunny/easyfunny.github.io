---
layout: post
title: Install Homebrew, on macos 13
categories: [Homebrew, macos]
description: Install Homebrew on macos 13
keywords: Homebrew, macos
---

# macos 安装Homebrew
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

# 安装 xctool
```
brew install xctool
```

# 安装OCLint、xcpretty
```
brew tap oclint/formulae
brew install oclint
sudo gem install xcpretty
brew update
brew upgrade oclint
```

# 安装 gcovr 
```
brew install gcovr
```



