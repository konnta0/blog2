---
{"dg-publish":true,"dg-home":false,"dg-metatags":{"og:title":"1 つのマシンで2つの git のユーザーを簡単に切り替える方法","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"1 つのマシンで2つの git のユーザーを簡単に切り替える方法","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"permalink":"/Engineering/1 つのマシンで2つの git のユーザーを簡単に切り替える方法/","metatags":{"og:title":"1 つのマシンで2つの git のユーザーを簡単に切り替える方法","og:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:card":"summary","twitter:title":"1 つのマシンで2つの git のユーザーを簡単に切り替える方法","twitter:image":"https://raw.githubusercontent.com/konnta0/blog2/refs/heads/main/konnta0.jpg","twitter:site":"@konnta0"},"dgPassFrontmatter":true,"created":"2024-12-08T12:07:32.343+09:00"}
---


#git #shell #github

## 使うモノ
ghq ... ローカルのリポジトリが管理が簡単にできる
direnv ... 指定のディレクトリ事に環境変数を設定できる

今回は private(個人開発用) と  business(仕事用) を切り替えることを想定します
また、動作検証などは ssh でのみ行っています

１．ghq を用いつつクローン時に設定を良い感じに書き換えるスクリプトの準備
`private-ghq`
```shell
#!/usr/bin/env ruby

github_host = "private.github.com"

username = "private-user" ## 適宜書き換える
email = "private-user@private.com" ## 適宜書き換える

if ARGV[0] == "get"
  args = []
  repo = ""

  ARGV[1..-1].each do |arg|
    repo = arg if arg.match(/^git@github.com/)
    args << arg.gsub(/^git@github.com/, "git@#{github_host}")
  end
  system "ghq", "get", *args
  exit_code = $?.to_i

  unless repo.empty?
    dir = repo.match(/^git@github.com:([^.]*)(\.git)?/)[1]
    ghq_root = `ghq root`.chomp

    Dir.chdir(File.join(ghq_root, "#{github_host}/#{dir}/")) do
      system "git config user.name '#{username}'"
      system "git config user.email #{email}"
    end
  end

  exit exit_code
else
  exec "ghq", *ARGV
end
```

`work-ghq`
```shell
#!/usr/bin/env ruby

github_host = "work.github.com" 

username = "work-user" ## 適宜書き換える
email = "work-user@work.co.jp" ## 適宜書き換える

if ARGV[0] == "get"
  args = []
  repo = ""

  ARGV[1..-1].each do |arg|
    repo = arg if arg.match(/^git@github.com/)
    args << arg.gsub(/^git@github.com/, "git@#{github_host}")
  end
  system "ghq", "get", *args
  exit_code = $?.to_i

  unless repo.empty?
    dir = repo.match(/^git@github.com:([^.]*)(\.git)?/)[1]
    ghq_root = `ghq root`.chomp

    Dir.chdir(File.join(ghq_root, "#{github_host}/#{dir}/")) do
      system "git config user.name '#{username}'"
      system "git config user.email #{email}"
    end
  end

  exit exit_code
else
  exec "ghq", *ARGV
end


```


２. `.ssh/config` の設定を追記
```config
Host work.github.com
    HostName github.com
    IdentityFile ~/.ssh/work
    User git
    AddKeysToAgent yes
    UseKeychain yes
    TCPKeepAlive yes
    IdentitiesOnly yes

Host private.github.com
    HostName github.com
    IdentityFile ~/.ssh/private
    User git
    AddKeysToAgent yes
    UseKeychain yes
    TCPKeepAlive yes
    IdentitiesOnly yes
```


３. clone をする
趣味アカウントで操作をしたい場合は `private-ghq` 
仕事アカウントで操作をしたい場合は`work-ghq`
を、利用して clone します
```shell
./private-ghq get git@github.com:konnta0/wabl-cs.git 
or
./work-ghq get git@github.com:company/repository.git 
```

４．ディレクトリによって git user を自動的に切り替えるようにする
ghq の設定はデフォルトと想定します。
これの設定を行う事によって指定のディレクトリ内に移動したときに自動でコミッターなどの環境変数が変更されます。
```
$ cd ~/ghq/private.github.com

$ cat << EOF > .envrc
export GIT_COMMITTER_NAME="private-user"
export GIT_COMMITTER_EMAIL="private-user0@private.com"
export GIT_AUTHOR_NAME="private-user"
export GIT_AUTHOR_EMAIL="private-user@private.com"
EOF

$ direnv allow
```

```
$ cd ~/ghq/work.github.com

$ cat << EOF > .envrc
export GIT_COMMITTER_NAME="work-user"
export GIT_COMMITTER_EMAIL="work-user0@work.com"
export GIT_AUTHOR_NAME="work-user"
export GIT_AUTHOR_EMAIL="work-user@work.com"
EOF
$ direnv allow
```