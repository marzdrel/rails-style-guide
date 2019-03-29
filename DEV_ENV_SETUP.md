# MAC OS Dev environment for rails development
============

- [ ] brew
- [ ] chrome - via browser or brew
- [ ] Xcode tools
- [ ] text-editor (sublime / atom / vim / other)
  Packages for sublime to smothe ruby / rails development: 
    - [ ] Package Control
    - [ ] SublimeLinter
    - [ ] AllAutoCompletes
    - [ ] BeautifyRuby
    - [ ] Emmet
    - [ ] BracketHighlighter
    - [ ] ProductiveSnippetsRuby
    - [ ] Ruby On Rails Snippets
- [ ] postgreapp with postico
- [ ] iTerm2 (with zsh and oh-my-zsh)
- [ ] [Spectacle](https://www.spectacleapp.com/)
- [ ] instal rvm / rbenv
- [ ] when using rbenv with zsh do:
    - `$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshenv`
    - `$ echo 'eval "$(rbenv init -)"' >> ~/.zshenv`
    - `$ echo 'source $HOME/.zshenv' >> ~/.zshrc`
    - `$ exec $SHELL`
- [ ] git configuration setting global user.name email etc. adding SSH key
- [ ] [redis via homebrew](https://medium.com/@petehouston/install-and-config-redis-on-mac-os-x-via-homebrew-eb8df9a4f298)
- [ ] GUI for git if using one (my pick [gitkraken](https://www.gitkraken.com/))
- [ ] [cheatsheetapp](https://www.cheatsheetapp.com/CheatSheet/)
- [ ] brew install libsodium
- [ ] brew install node (for NPM)
- [ ] gem install hirb
    - in ~/.irbc (create file if it does not exist) add:
	```ruby
	if defined? Rails
  	  begin
    	    require 'hirb'
    	    Hirb.enable
  	  rescue LoadError
  	  end
	end
	```
