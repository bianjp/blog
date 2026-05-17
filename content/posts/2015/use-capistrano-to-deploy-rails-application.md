---
title: 使用 Capistrano 部署 Rails 应用
date: 2015-11-08T18:24:40+08:00
tags: ["capistrano", "rails"]
draft: false
---

Capistrano 是一个远程服务器自动化工具，使用 Ruby 编写。 它是一个通用类型的工具，并非专为 Rails 设计，但对 Rails 支持很好。

<!--more-->

## 优点

* 无需对服务器进行任何配置
* 一键部署、回滚
* 易于扩展，可方便地添加自定义任务

## 基本原理

预先定义一系列任务（本质上都是 shell 脚本），然后通过 ssh在 远程服务器上执行任务。 使用钩子定义任务的执行时间、顺序。 可创建多套部署方案，capistrano 中称为 stage，每个 stage 有一个单独的配置文件。

## 基本流程

假定服务器上使用 Nginx + Passenger + Rvm 运行 Rails 应用，具体安装配置参见各软件文档。

部署 Rails 应用的基本流程是：

1.  将代码推送到远程仓库
2.  本地执行 capistrano 部署命令
3.  capistrano 通过 ssh 连接远程服务器，从远程代码仓库拉取代码，然后执行安装依赖、编译资源文件、迁移数据库等任务
4.  重启 passenger 使新代码生效

## 基本要求

### 服务器

* 有一个部署用的用户，该用户应对部署目录有读写权限，且有执行rvm, passenger-config等命令的权限
* 可连接远程代码仓库（不要求部署用户有拉取权限）

### 本地

* 能通过ssh密钥以部署用户身份登录远程服务器
* 对远程代码仓库有拉取权限

## 安装

只需在本地安装 capistrano 在项目的 Gemfile 中加入以下内容：

```ruby
group :development do
  # deploy
  gem 'capistrano-rails', '~> 1.1.5'
  gem 'capistrano-rvm'
end

```

执行 `bundle install` 安装新依赖

## 本地初始化

```shell
bundle exec cap install STAGES=development,production
```

初始化后生成以下文件结构

```text
├── Capfile                # 控制插件、任务的加载
├── config
│   ├── deploy             # 不同stage的配置文件
│   │   ├── development.rb
│   │   └── production.rb
│   └── deploy.rb          # 全局配置，各stage共用
└── lib
    └── capistrano
            └── tasks      # 自定义任务的存放位置
```

若初始化时不指定 STAGES，默认创建 staging, production 两个 stage。

## 配置

1. 加载 rails, rvm 插件，在 `Capfile` 中加入:
    ```ruby
    require 'capistrano/rails'
    require 'capistrano/rvm'
    ```

2. 修改全局配置 `config/deploy.rb`
    ```ruby
    # 应用名称
    set :application, 'liveneeq'

    # 远程仓库地址
    set :repo_url, 'git@git.thecampus.cc:yang/liveneeq2.git'

    # 服务器端部署目录。可使用fetch(:xxx)获取已设置的变量
    set :deploy_to, "/data/app/www/#{fetch(:application)}"

    # 在不同版本间共享的文件
    set :linked_files, fetch(:linked_files, []).push('config/database.yml', 'config/secrets.yml')

    # 在不同版本间共享的目录
    set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 
        'vendor/bundle', 'public/system', 'public/uploads')

    # ruby版本
    set :rvm_ruby_version, '2.2.2'

    # 默认Rails.env会等于stage的名称而非始终production，导致代码中判断模式错误。此处强制所有stage都为production模式
    set :stage, :production

    ```

3.  修改各个 stage 的配置 我们使用 development, production 两个 stage，development 部署 dev 分支到开发/测试服务器，production 部署 master 分支到生产服务器

    ```ruby
    # config/deploy/development.rb:

    # 远程服务器的地址、用户、角色
    # 可配置多台服务器
    server 'IP', user: 'USER_NAME', roles: %w(app db web)

    # 要部署的分支
    set :branch, 'dev'
    ```

    ```ruby
    # config/deploy/production.rb

    # 远程服务器的地址、用户、角色
    # 可配置多台服务器
    server 'IP', user: 'USER_NAME', roles: %w(app db web)

    # 要部署的分支
    set :branch, 'master'
    ```

## 服务器端初始化

1.  创建部署目录
2.  在部署目录下创建 shared 子目录，里面添加 `config/deploy.rb` 中配置的 linked_files
3.  创建数据库

## Capistrano 命令

```shell
# 查看可用任务
bundle exec cap -T

# 部署
bundle exec cap STAGE_NAME deploy

# 回滚
bundle exec cap STAGE_NAME deploy:rollback
```

*   执行任何任务都需要指定stage名称
*   使用`bundle exec`执行cap命令以避免不可预知的风险

## 自定义任务

### 集成 bower

若项目中使用 bower 管理前端依赖，需自行添加任务安装 bower 依赖 创建 `lib/capistrano/tasks/bower.rake` 文件，内容如下：

```ruby
namespace :bower do
  desc 'Install bower dependencies'
  task :install do
    on roles(:web) do |host|
      within release_path do
        # 安装依赖
        execute :bower, :install
        # 清理无用包
        execute :bower, :prune
        info "Bower dependencies installed successfully on #{host}"
      end
    end
  end
end

```

然后在 `config/deploy.rb` 末尾加入

```ruby
before :'deploy:compile_assets', :'bower:install'
```

### 部署后重启 passenger

部署新版本后需要重启 passenger 才能使新代码生效，可通过 [capistrano-passenger](https://github.com/capistrano/passenger) 插件实现部署完成后自动重启，也可自行添加一个重启任务： 创建 `lib/capistrano/tasks/restart.rb`，内容如下：

```ruby
namespace :passenger do
  desc 'Restart passenger app'
  task :restart do
    on roles(:app), in: :sequence, wait: 5, limit: 2 do |host|
      # 若使用的是passenger企业版，可添加参数--rolling-restart以实现无缝重启
      execute :'passenger-config', 'restart-app', fetch(:deploy_to), '--ignore-app-not-running'
      info "Restart passenger successfully on #{host}"
    end
  end
end
```

然后在 `conig/deploy.rb` 末尾加入

```ruby
after :'deploy:publishing', :'passenger:restart'
```

## 不足之处

1.  非完全自动化，只能说是半自动，因为需要本地执行部署命令
2.  完全依赖本地环境，配置在本地，执行命令在本地，而且要求本地用户能ssh登录服务器

曾经使用 git hook + shell 脚本实现过一种部署方案，指定的分支有更新（主动推送到远程仓库或在 GitLab 上接受合并请求）时就自动部署。 完全自动化、完全不依赖本地环境，但服务器端的配置略麻烦。 两种方案各有优劣，或许结合一下会更好？
