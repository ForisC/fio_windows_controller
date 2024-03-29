def remote = [:]
remote.name = 'node-1'
remote.host = params.client
remote.allowAnyHosts = true
currentBuild.description = env.Description

node {
    withCredentials(
        [
            usernamePassword(
                credentialsId: 'windows_client',
                usernameVariable: 'USERNAME',
                passwordVariable: 'PASSWORD'
            )
        ]
    ) {
        remote.user = USERNAME
        remote.password  = PASSWORD
        stage('Env Preparing') {
            git branch: 'main', url: 'https://github.com/foris323/fio_windows_controller.git'
            sshCommand remote: remote, command: 'taskkill /f /T /IM fio.exe || exit /b 0'
        }

        stage('FIO random read') {
            command = "fio -filename=${env.drive}:\\fio -direct=1 -iodepth 1" +
            ' -thread -rw=randread -ioengine=windowsaio' +
            " -bs=16k -size=${env.TestSize} -numjobs=30 -runtime=1000 -group_reporting" +
            ' -name=randread --output=randread.json --output-format=json'
            sshCommand remote: remote, command: command
        }
        stage("FIO random write") {
            command = "fio -filename=${env.drive}:\\fio -direct=1 -iodepth 1" +
            ' -thread -rw=randwrite -ioengine=windowsaio' +
            " -bs=16k -size=${env.TestSize} -numjobs=30 -runtime=1000 -group_reporting" +
            ' -name=randwrite --output=randwrite.json --output-format=json'
            sshCommand remote: remote, command: command
        }
        stage("FIO sequential read") {
            command = "fio -filename=${env.drive}:\\fio -direct=1 -iodepth 1" +
            ' -thread -rw=read -ioengine=windowsaio' +
            " -bs=16k -size=${env.TestSize} -numjobs=30 -runtime=1000 -group_reporting" +
            ' -name=read --output=read.json --output-format=json'
            sshCommand remote: remote, command: command
        }
        stage("FIO sequential write") {
            command = "fio -filename=${env.drive}:\\fio -direct=1 -iodepth 1" +
            ' -thread -rw=write -ioengine=windowsaio' +
            " -bs=16k -size=${env.TestSize} -numjobs=30 -runtime=1000 -group_reporting" +
            ' -name=write --output=write.json --output-format=json'
            sshCommand remote: remote, command: command
        }
        stage('Collect data') {
            sshGet remote: remote, from: 'randread.json', into: 'randread.json', override: true
            sshGet remote: remote, from: 'randwrite.json', into: 'randwrite.json', override: true
            sshGet remote: remote, from: 'read.json', into: 'read.json', override: true
            sshGet remote: remote, from: 'write.json', into: 'write.json', override: true
            sh 'cat randread.json'
            sh 'cat randwrite.json'
            sh 'cat read.json'
            sh 'cat write.json'
            sh 'python3 gen_csv.py'
            plot csvSeries: [[
                            file: 'data.csv',
                            exclusionValues: '',
                            displayTableFlag: false,
                            inclusionFlag: 'OFF',
                            url: '']],
            csvFileName: "plot-iops.csv",
            group: 'Plot Group',
            title: 'FIO test',
            style: 'line',
            exclZero: true,
            keepRecords: false,
            logarithmic: false,
            numBuilds: '10',
            useDescr: true,
            yaxis: 'iops',
            yaxisMaximum: '',
            yaxisMinimum: ''
        }
    }
}