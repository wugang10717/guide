# python-developer

1: Environment Setup:

   install virtualenv and create new environment
   
     $pip install virtualenv
     ### create python 2.7 env
     $mkdir /home/gwu/virtualenv
     $cd /home/gwu/virtualenv
     $virtualenv -p /usr/bin/python2.7 --no-site-packages python2.7
     $source /home/gwu/eos/python2.7/bin/activate
     $mkdir /home/gwu/eos_2.7
     $cd /home/gwu/eos_2.7
     install need packages and start code in python2.7 env
     
   add alias to your shell
   
     alias eos2="source /home/gwu/eos/python2.7/bin/activate; cd /home/gwu/eos_2.7; export PYTHONPATH=/home/gwu/eos_2.7"
     
   
   install virtualenv and create new environment
     
     ### create python 3.5 env
     $cd /home/gwu/virtualenv
     $virtualenv -p /usr/bin/python3.5 --no-site-packages python3.5
     $source /home/gwu/eos/python3.5/bin/activate
     $mkdir /home/gwu/eos_3.5
     $cd /home/gwu/eos_3.5
     install need packages and start code in python3.5 env
   
   add alias to your shell
   
     alias eos3="source /home/gwu/eos/python3.5/bin/activate; cd /home/gwu/eos_3.5; export PYTHONPATH=/home/gwu/eos_3.5"
   
     
   exit current env
   
     $deactivate 
     


2: python code unitest, linting, and coverage 

   install packages:
   
    pip install --upgrade pytest pytest-pylint pytest-cov pytest-xdist pytest-flakes

   Set up the following aliases in your shell:
   
    alias pyflow-test='py.test --color=yes -s --verbose -f'
    alias pyflow-cov='py.test -f --color=yes --cov --cov-report term-missing'
    alias pyflow-lint='py.test --color=yes -f --pylint'
    alias pyflow-flakes='py.test --color=yes -f --flakes'
    
    
3:   