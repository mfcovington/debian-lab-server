Installation directions for RadiantCMS

Install ruby  
`sudo apt-get install ruby`

Download RubyGems zip file from: http://rubygems.org/pages/download

Unzip the RubyGems zip file, then `cd` into the directory and run `ruby setup.rb`

Install the Ruby dev package  
`apt-get install ruby1.9.1-dev`

Now finally, install RadiantCMS  
`gem install radiant`

Need to install sqlite too  
`sudo apt-get install sqlite`

Need to install a newer version of ruby (2.3.1 not 1.9.3)
`get -O ruby-install-0.5.0.tar.gz https://github.com/postmodern/ruby-install/archive/v0.5.0.tar.gz`  
`tar -xzvf ruby-install-0.5.0.tar.gz`  
`cd ruby-install-0.5.0/`  
`sudo make install`  
`ruby-install ruby`  
