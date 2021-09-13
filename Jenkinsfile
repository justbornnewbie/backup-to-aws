pipeline {
    agent {
        label "worker1"
    }
    stages {
        stage ("take backup") {
            steps {
                sh '''
                d=$(date +%d-%m-%y-%T)
                echo $d > date.txt
                myd=$(cat date.txt)
                mysqldump -u root -h 172.18.0.2 -proot testdb --column-statistics=0 > /opt/mysql-backup.$myd.sql
                '''
            }
        }
        stage ("Copy to AWS") {
            steps {
                sh '''
                myd=$(cat date.txt)
                aws s3 cp /opt/mysql-backup.$myd.sql s3://lelebey/mysql-backup.$myd.sql
                '''
            }            
        }
        stage ("file retention") {
            steps {
                sh '''
                myd=$(cat date.txt)
                #ls -t | tail -n +2 | xargs rm -- #this will delete all the files except newest 2
                aws s3 ls s3://lelebey/ --recursive | sort -k1 | sort -k2 | head -n -3 | awk '{$1=$2=$3=""; print $0}' | sed 's/^[ \t]*//' | while read -r line ; do
                    echo "Removing \"${line}\"";
                    aws s3 rm "s3://lelebey/${line}";
                done
                '''
            }
        }
    }
    post {
        always {
            emailext body: 'A Test EMail', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
        }
    }
}