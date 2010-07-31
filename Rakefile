namespace :deploy do
  task :restart_passenger do
    puts "restarting Passenger web server"
    Dir.chdir(File.expand_path(File.join(File.dirname(__FILE__), 'tmp')))
    system("touch restart.txt")
  end

  task :post_deploy => [ :restart_passenger ]
end

