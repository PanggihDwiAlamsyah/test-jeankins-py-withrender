node { 
    properties([
        pipelineTriggers([
            pollSCM('H/2 * * * *')  // Poll setiap 2 menit
        ])
    ])

    def venvPath = 'venv'

    try {
        stage('Checkout') {
            echo "Mengambil kode terbaru dari repository Git"
            checkout scm
        }

        stage('Install Dependencies') {
            echo "Menginstal dependencies sistem yang dibutuhkan"
            sh '''#!/bin/bash
                sudo apt update
                sudo apt install -y python3.11 python3.11-dev python3.11-venv python3-pip build-essential
            '''

            echo "Membuat ulang virtual environment"
            sh '''#!/bin/bash
                rm -rf venv
                python3 -m venv venv
            '''

            echo "Menginstal pytest dan pyinstaller tanpa aktivasi virtual environment"
            sh '''#!/bin/bash
                venv/bin/pip install --upgrade pip
                venv/bin/pip install pytest pyinstaller
            '''
        }

        stage('Build') {
            echo "Menjalankan py_compile menggunakan Python dari virtual environment"
            sh '''#!/bin/bash
                venv/bin/python -m py_compile sources/add2vals.py sources/calc.py
            '''

            echo "Stashing hasil compile"
            stash name: 'compiled-results', includes: 'sources/*.py*'
        }

        stage('Test') {
            echo "Menjalankan pytest menggunakan Python dari virtual environment"
            sh '''#!/bin/bash
                venv/bin/pytest --junit-xml test-reports/results.xml sources/test_calc.py
            '''

            echo "Menyimpan hasil uji"
            junit 'test-reports/results.xml'
        }

        stage('Manual Approval') {
            echo "Menunggu persetujuan untuk tahap Deploy"
            def userInput = input(
                message: 'Lanjutkan ke tahap Deploy?',
                parameters: [
                    choice(name: 'Continue', choices: ['Proceed', 'Abort'], description: 'Pilih untuk melanjutkan atau menghentikan pipeline')
                ]
            )

            if (userInput == 'Abort') {
                error('Pipeline dihentikan oleh pengguna.')
            }
        }

        stage('Deploy') {
            echo "Menjalankan aplikasi"
            sh '''#!/bin/bash
                nohup venv/bin/python sources/app.py &
            '''

            echo "Menunggu selama 1 menit agar aplikasi tetap berjalan"
            sh 'sleep 60'

            echo "Deploy selesai, aplikasi akan dihentikan."
        }

    } catch (Exception err) {
        echo "Build failed: ${err}"
        currentBuild.result = 'FAILURE'
    }
}
