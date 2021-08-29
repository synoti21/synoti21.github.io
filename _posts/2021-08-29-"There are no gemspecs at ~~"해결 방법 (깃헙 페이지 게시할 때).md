---
title: "2021-08-29-깃허브 페이지 게시 - Jekyll 관련 에러 해결 방법"
excerpt: "개발 블로그 제작"
date : 2021-08-29
last_modified_at : 2021-08-29

---
## 하루종일 삽질한 이야기..
오늘 깃허브 블로그 페이지를 만들다가 로컬 페이지로 호스팅을 할려고 bundle exec jekyll serve를 실행할려 했는데, 계속
~~~shell
asonahn@anjiwan-ui-MacBookAir synoti21.github.io % bundle exec jekyll serve

[!] There was an error parsing `Gemfile`: There are no gemspecs at /Users/jasonahn/Desktop/synoti21.github.io. Bundler cannot continue.

 #  from /Users/jasonahn/Desktop/synoti21.github.io/Gemfile:2
 #  -------------------------------------------
 #  source "https://rubygems.org"
 >  gemspec
 #  -------------------------------------------
 ~~~
 이렇게 뜨면서 호스팅이 실패했다고 떴다. 처음에는 지킬을 안깔았나 싶어서 ruby에서 Jekyll을 설치하고, 최신 버전의 ruby를 새로 깔았는데 여전히 똑같은 문제가 뜨면서 호스팅이 되지 않았다.
 설상가상으로 jekyll을 설치하는 와중에도 Bus error이 뜨면서 bundler가 설치가 안되는 문제까지 생겼다. 오늘 해결할 문제는 총 세가지다
 * Bus Error 
 * webrick (Load Error)
 * There are no gemspecs at ~~~
 ------
 ### 해결 방안(1) - Bus Error
 Bus Error은 jekyll 관련 명령어를 사용할 때 나타날 수 있다. 나의 경우, jekyll 버전을
 확인하려고 명령어를 칠 때 에러가 났다. 
 원인을 보니 우선 내가 쓰는 노트북이 맥북 M1이었던 점이 있었다. 구글에 검색해보니 m1칩을 사용하는 맥북 이용자들도 bus error이 뜨면서 충돌했다고 한다. 
![](https://synoti21.github.io/assets/images/crash.png)
이런 에러창이 터미널에 뜨면서 실행이 안된다면 Ruby 문제일 가능성이 높다. 맥북에는 리눅스나
다른 운영체제와 달리 Ruby가 미리 설치되어 있으며, 이 Ruby는 맥북 시스템 용으로 설정되어 있기 때문에 우리 같은 사용자가 사용할 경우 에러를 일으킬 수 있다. 따라서 맥북 M1사용자는 루비를 따로 설치 해야 한다.

####1. Homebrew 설치
깃헙 페이지를 만들기 위해 블로그를 보면 apt를 이용해 ruby를 설치하는 경우가 있지만, 맥북은 apt명령어 대신 brew명령어를 사용해야 한다. <https://brew.sh>를 통해 Homebrew를 설치한다.

####2. rbenv 설치
ruby 버전 관리자 rbenv를 통해 ruby를 새로 설치한다. 
~~~shell
brew update
brew install rbenv ruby-build
~~~
설치했을 경우 버전을 확인해보면,
~~~shell
rbenv versions
~~~
다음과 같이 출력될 것이다
~~~shell
* system (set by /Users/jasonahn/.rbenv/version)
~~~
이 system은 위에서 말했듯 맥북 시스템이 사용하는 미리 설치된 ruby이기 때문에 우리가 사용할 수 있는 ruby를 따로 설치해줘야 한다.
~~~shell
rbenv install -l
rbenv install 3.0.2
~~~
첫번째 줄의 명령어를 입력하면 설치 가능한 Ruby 버전들이 나올텐데, 이때 가장 최신버전을 설치해주면 된다. 21.8.29 기준으로 최신버전인 3.0.2를 설치하자.
그 후, 다시 버전을 확인해 보면 다음과 같이 나올 것이다.
~~~shell
* system
  3.0.2 (set by /Users/jasonahn/.rbenv/version)
~~~
새로운 Ruby가 설치되었으나 여전히 system상의 Ruby를 사용중이므로 3.0.2로 전환해야 한다.
~~~shell
rbenv global 3.0.2
~~~
그러면 글로벌 버전이 바뀐 것을 볼 수 있다.
~~~shell
  system
* 3.0.2 (set by /Users/jasonahn/.rbenv/version)
~~~~
주소를 보면 .rbenv/version으로 되어있으므로 PATH에 추가해야할 필요가 있다.
~~~shell
vim ~/.zshrc
~~~
.zshrc에 다음 내용을 추가한 뒤 저장한다.
~~~shell
[[ -d ~/.rbenv  ]] && \
  export PATH=${HOME}/.rbenv/bin:${PATH} && \
  eval "$(rbenv init -)"
~~~
그후 변경사항을 source를 통해 적용시킨다.
~~~shell
source ~/.zshrc
~~~
그후에 다시 jekyll bundler를 설치하고 버전을 확인해보면 bus error없이 정상적으로 출력될 것이다.
~~~
gem install jekyll bundler
jekyll -v
~~~
만약 이렇게 해도 bus error이 출력이 되지 않는다면 해당 문제를 일으키는 gem을 다시 설치하는 방법도 있다.
~~~shell
gem uninstall sassc
gem install sassc -- --disable-march-tune-native
gem uninstall ffi
gem install ffi
bundle update --bundler
~~~
그후 본인의 깃헙 페이지 repository로 가서 누락된 bundle을 설치 해주면 된다.
~~~shell
bundle install
~~~

### 해결방안(2) - webrick error
간혹 깃헙 페이지를 로컬 호스팅 할 때 다음과 같이 에러메시지가 출력되면서 실행이 안되는 경우가 있다.
~~~shell
bundler: failed to load command: jekyll (/Users/jasonahn/.rbenv/versions/3.0.2/bin/jekyll)
/Users/jasonahn/.rbenv/versions/3.0.2/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
~~~
이 경우는 ruby버전이 3.0.0이상 부터 jekyll이 실행이 되지 않는 문제이기 때문에 webric을 수동으로 설치해주면 된다.
~~~shell
bundle add webrick
~~~
그러면 해결될 것이다.

### 해결방안(3) - "There are no gemspecs at..."
2번까지 모두 완료 했는데도 아래와 같은 에러메시지가 출력될 수 있다.
~~~shell
[!] There was an error parsing `Gemfile`: There are no gemspecs at /Users/jasonahn/Desktop/synoti21.github.io. Bundler cannot continue.

 #  from /Users/jasonahn/Desktop/synoti21.github.io/Gemfile:2
 #  -------------------------------------------
 #  source "https://rubygems.org"
 >  gemspec
 #  -------------------------------------------
~~~
이는 본인이 만들고자 하는 깃헙 페이지 repository안에 있는 gemfile이 미완성된 상태로 존재하기 때문에 직접 수정해야 한다.
처음 gemfile을 열면 이런 상태로 있을 것이다.
~~~txt
source "https://rubygems.org"
gemspec
~~~
이 내용을 다음과 같이 변경한다.
~~~txt
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

gem "tzinfo-data"
gem "wdm", "~> 0.1.0" if Gem.win_platform?

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
  gem "jekyll-include-cache"
  gem "jekyll-algolia"
end
gem "webrick", "~> 1.7"
~~~

