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

    stage('Backing Up the remote server') { 
      environment {
        dir='/home/centos/alpha'
        Path1='/home/centos/alpha/deploy1'
        Path2='/home/centos/alpha/deploy2'
        Path3='/home/centos/alpha/deploy3'  
        Backup1='/home/centos/alpha/backup1'
        Backup2='/home/centos/alpha/backup2'
        Backup3='/home/centos/alpha/backup3'
        Name1='$(ls -v /Path | grep [i]ndex.js | tail -1)'
        Name2='$(ls -v /Path | grep [i]ndex2.js | tail -1)'
        Name3='$(ls -v /Path | grep [i]ndex3.js | tail -1)'
      }
      steps { 
          sh '''
              ssh centos@192.168.231.144 "
              cd $dir
              mkdir backup1 backup2 backup3
              if [ -f "${env.Path1}"/"${env.Name1}"]; then
                echo -e '${env.Name1} file exist'
                mv ${env.Path1}/${env.Name1} ${env.Backup1}
              else
                echo -e "file doesn't exist"
              fi
              echo -e 'backup done for ${env.Name1} file.'
              if [ -f "${env.Path2}/${env.Name2}" ]; then
                echo -e '${env.Name2} file exist'
                mv ${env.Path2}/${env.Name2} ${env.Backup2}
              else
                echo -e "file doesn't exist"
              fi
              echo -e 'backup done for ${env.Name2} file.'
              if [ -f "${env.Path3}/${env.Name3}" ]; then
                echo -e '${env.Name3} file exist'
                mv ${env.Path3}/${env.Name3} ${env.Backup3}
              else
                echo -e "file doesn't exist"
              fi
              echo -e 'backup done for ${env.Name3} file.'"
     '''
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
              exit 0;
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
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1
            cd /home/centos/alpha/deploy1
            node index.js > /dev/null 2>&1 <&- &
            echo -e 'stage 1 : rollback complete...................................................................'
            exit 1;
ENDSSH 
      '''
        }
        failure { 
          echo 'The state is Failure..........'
          echo 'Rollback back to Original......'
          sh '''
            ssh -T centos@192.168.231.144 << ENDSSH
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1
            cd /home/centos/alpha/deploy1
            node index.js > /dev/null 2>&1 <&- &
            echo -e 'stage 1 : rollback complete...................................................................'
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
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo kill -9 $(lsof -i:3002)
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*   
            cd /home/centos/alpha/deploy2
            node index2.js > /dev/null 2>&1 <&- &
            echo -e 'stage 2 : rollback complete...................................................................'
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
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo kill -9 $(lsof -i:3002)
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*   
            cd /home/centos/alpha/deploy2
            node index2.js > /dev/null 2>&1 <&- &
            echo -e 'stage 2 : rollback complete...................................................................'
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
                    node index.js > /dev/null 2>&1 <&- &  "
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
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo kill -9 $(lsof -i:3002)
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*            
            mv /home/centos/alpha/backup3/index3.js /home/centos/alpha/deploy3/
            sudo rm -rf /home/centos/alpha/backup3/*
            cd /home/centos/alpha/deploy3
            node index3.js > /dev/null 2>&1 <&- &
            echo -e 'stage 3 : rollback complete...................................................................'
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
            mv /home/centos/alpha/backup1/index.js /home/centos/alpha/deploy1/
            sudo rm -rf /home/centos/alpha/backup1/*
            sudo kill -9 $(lsof -i:3002)
            mv /home/centos/alpha/backup2/index2.js /home/centos/alpha/deploy2/
            sudo rm -rf /home/centos/alpha/backup2/*            
            mv /home/centos/alpha/backup3/index3.js /home/centos/alpha/deploy3/
            sudo rm -rf /home/centos/alpha/backup3/*
            cd /home/centos/alpha/deploy3
            node index3.js > /dev/null 2>&1 <&- &   
            echo -e 'stage 3 : rollback complete...................................................................' 
            exit;
ENDSSH 
      '''
         }
       }
     }
  }
}
