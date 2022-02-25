pipeline {
  agent any
    
  tools {nodejs "node"}
    
  stages {

    stage ('Prepare destination host') {
      steps {
        sh '''
          ssh -T centos@192.168.231.144 << ENDSSH
          cd /home/centos
          sudo rm -rf alpha/
          sudo sleep 2
          mkdir alpha
          sudo chmod +x alpha/
          cd alpha/
          mkdir deploy1 deploy2 deploy3
          sudo dnf install npm -y
          sudo npm install
ENDSSH
      '''
      }
    }

    stage('checkout SCM') {
      steps {
        git 'https://github.com/bhimra/alpha-deployment.git'
        sh 'pwd && sudo chmod +x .'
      }
    }

    stage('SCP application on /locations') {
      steps {
        sh 'scp index.js centos@192.168.231.144:/home/centos/alpha/deploy1'
        sh 'scp index2.js centos@192.168.231.144:/home/centos/alpha/deploy2'
        sh 'scp index3.js centos@192.168.231.144:/home/centos/alpha/deploy3'
        sh 'echo "All deployment folder are READY."'
      }
    }   

    stage ('Kill existing node service') {
      steps {
        sh '''
          ssh -t -t  centos@192.168.231.144 'bash -s << 'ENDSSH'
          if [[ Z=$(sudo ps aux | grep -i [n]ode | awk 'NR==1' | gawk {'print $2'}) ]];
          then
              sudo kill -9 $Z
              echo "node service stop successfully."
              sudo pkill node
              exit 0
          else
              echo "node service failed"
          fi
ENDSSH'
       '''
       }
    }
    stage ('Stage - 1: Build & Deploy index.js service') {
      steps {
        sh '''
        set -x
        ssh centos@192.168.231.144 "
                    cd /home/centos/alpha/deploy1
                    node index.js > /dev/null 2>&1 <&- & 
                    mkdir /home/centos/alpha/backup1
                    cp index.js /home/centos/alpha/backup1/"
        sudo sleep 3                    
        X=$(curl -k  -o /dev/null -s -w %{http_code} http://192.168.231.144:3000)
        if [ $X -eq 200 ];
            then
                echo -e 'web site on port 3000 is running'
            else
                echo -e 'web site on port 3000 is down' 
        fi '''
       }
      post {
        unstable {
          echo 'Deployment Unstable..........'
          echo 'Rollback back to Previous state......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            echo -e 'index.js service stop & port 3000 is down'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1'
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1
            exit 1;
ENDSSH 
      '''
        }
        failure {
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            echo -e 'index.js service stop & port 3000 is down'
            sudo rm -rf /home/centos/alpha/deploy1/* 
            echo -e 'removed index.js from deploy1'
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1
            exit;
ENDSSH 
      '''
        }
      }
    }
    stage ('Stage - 2: Build & Deploy index2.js service') {
      steps {
        sh '''
        set -x
        ssh centos@192.168.231.144 "
                    cd /home/centos/alpha/deploy2
                    node index2.js > /dev/null 2>&1 <&- & 
                    mkdir /home/centos/alpha/backup2
                    cp index2.js /home/centos/alpha/backup2/ "
        sudo sleep 3                    
        X=$(curl -k  -o /dev/null -s -w %{http_code} http://192.168.231.144:3002)
        if [ $X -eq 200 ];
            then
                echo -e 'web site on port 3002 is running'
            else
                echo -e 'web site on port 3002 is down' 
        fi '''
       }
      post {
        unstable {
          echo ' Deployment Unstable..........'
          echo 'Rollback back to previous state......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            sudo kill -9 $(lsof -i:3000)
            sudo rm -rf /home/centos/alpha/deploy1/*
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo rm -rf /home/centos/alpha/deploy2/*
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*
            exit 1;
ENDSSH 
      '''
        }
        failure {
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            sudo kill -9 $(lsof -i:3000)
            sudo rm -rf /home/centos/alpha/deploy1/*
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo rm -rf /home/centos/alpha/deploy2/*
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*
            exit;
ENDSSH 
      '''
        }
      }
    }
    
    stage ('Stage - 3: Build & Deploy index3.js service') {
      steps {
        sh '''
        set -x
        ssh centos@192.168.231.144 "
                    cd /home/centos/alpha/deploy3
                    node index3.js > /dev/null 2>&1 <&- &
                    mkdir /home/centos/alpha/backup3
                    cp index3.js /home/centos/alpha/backup3/  "
        sudo sleep 5                    
        X=$(curl -k  -o /dev/null -s -w %{http_code} http://192.168.231.144:3003)
        if [ $X -eq 200 ];
            then
                echo -e 'web site on port 3003 is running'
            else
                echo -e 'web site on port 3003 is down' 
        fi '''
       }
      post {
        unstable {
          echo 'The state is Unstable..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            sudo kill -9 $(lsof -i:3000)
            sudo rm -rf /home/centos/alpha/deploy1/*
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo kill -9 $(lsof -i:3002)
            sudo rm -rf /home/centos/alpha/deploy2/*
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*            
            sudo rm -rf /home/centos/alpha/deploy3/*
            mv /home/centos/alpha/backup2/index3.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup3/*            
            exit 1;
ENDSSH 
      '''
        }
        failure {
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            sudo kill -9 $(lsof -i:3000)
            sudo rm -rf /home/centos/alpha/deploy1/*
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo kill -9 $(lsof -i:3002)
            sudo rm -rf /home/centos/alpha/deploy2/*
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*            
            sudo rm -rf /home/centos/alpha/deploy3/*
            mv /home/centos/alpha/backup2/index3.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup3/*            
            exit;
ENDSSH 
      '''
         }
       }
     }
  }
}
