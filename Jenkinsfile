pipeline {
    agent any
    environment {
        HDFS_URL = "http://sber-hadoop.tagobar.corp:50070"
        YARN_URL = "http://sber-hadoop.tagobar.corp:8088/cluster"
        HBASE_URL = "http://sber-hadoop.tagobar.corp:60010/master-status"
        
        HDFS_STATUS = false
        YARN_STATUS = false
        HBASE_STATUS = false
        ALL_PASS = false
        
        HDFS_SUBSTRING = "Hadoop Administration"
        YARN_SUBSTRING = "All Applications"
        HBASE_SUBSTRING = "Master:"
        
        NGINX_URL = "http://sber-hadoop.tagobar.corp"
        NGINX_SUBSTRING = "Welcome to <strong>nginx</strong>"
    }
    stages {
        stage('Check for Hadoop components status') {
            steps {
                script {
                    echo "=== Check for HDFS service status ==="
                    final String response = sh(script: "curl -s ${HDFS_URL} || exit 0", returnStdout: true).trim()
                    if (response.indexOf("${HDFS_SUBSTRING}") >=0) {
                        echo "=== HDFS status: OK ==="
                        HDFS_STATUS = true
                    } else {
                        echo "=== HDFS status: Fail ==="
                    }
                }
                script {
                    echo "=== Check for Yarn service status ==="
                    final String response = sh(script: "curl -s ${YARN_URL} || exit 0", returnStdout: true).trim()
                    if (response.indexOf("${YARN_SUBSTRING}") >=0) {
                        echo "=== YARN status: OK ==="
                        YARN_STATUS = true
                    } else {
                        echo "=== YARN status: Fail ==="
                    }
                }
                script {
                    echo "=== Check for HBase service status ==="
                    final String response = sh(script: "curl -s ${HBASE_URL} || exit 0", returnStdout: true).trim()
                    if (response.indexOf("${HBASE_SUBSTRING}") >=0) {
                        echo "=== HBase status: OK ==="
                        HBASE_STATUS = true
                    } else {
                        echo "=== HBase status: Fail ==="
                    }
                }
                script {
                    if (HDFS_STATUS == true && YARN_STATUS == true && HBASE_STATUS == true) {
                        echo "=== All checks pass ==="
                        ALL_PASS = true
                    } else {
                        echo "=== Checks failed, exiting... ==="
                        sh 'exit 1'
                    }
                }
            }
        }
        stage('Install Nginx') {
            when {
                expression {
                    ALL_PASS == true
                }
            }
            steps {
                ansiblePlaybook([
                        inventory   : 'hosts.inv',
                        playbook    : 'nginx.yml',
                        installation: 'ansible',
                        colorized   : true,
                        tags: "install-nginx",
                ])
                script {
                    echo "=== Check for Nginx welcome page ==="
                    final String response = sh(script: "curl -s ${NGINX_URL} || exit 0", returnStdout: true).trim()
                    if (response.indexOf("${NGINX_SUBSTRING}") >=0) {
                        echo "=== Nginx status: OK ==="
                    } else {
                        echo "=== Nginx status: Fail ==="
                        sh 'exit 1'
                    }
                }
            }
        }
    }
}