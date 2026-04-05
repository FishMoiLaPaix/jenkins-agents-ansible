final String cronExpr = env.BRANCH_IS_PRIMARY ? 'H/30 * * * *' : ''

properties([
    buildDiscarder(logRotator(numToKeepStr: '10')),
    disableConcurrentBuilds(abortPrevious: true),
    pipelineTriggers([cron(cronExpr)]),
])

node('jnlp-linux-amd64') {
    timeout(time: 15, unit: 'MINUTES') {
        checkout scm

        stage('Install Dependencies') {
            sh 'pip install --user ansible-lint yamllint'
            sh 'export PATH=$PATH:$HOME/.local/bin && yamllint --version && ansible-lint --version'
        }

        stage('YAML Lint') {
            sh '''
                export PATH=$PATH:$HOME/.local/bin
                yamllint --strict .
            '''
        }

        stage('Ansible Lint') {
            sh '''
                export PATH=$PATH:$HOME/.local/bin
                ansible-lint playbooks/
            '''
        }

        stage('Ansible Syntax Check') {
            sh '''
                export PATH=$PATH:$HOME/.local/bin
                for playbook in playbooks/*.yml; do
                    echo "Checking syntax: ${playbook}"
                    ansible-playbook --syntax-check "${playbook}"
                done
            '''
        }

        stage('Dry Run (Check Mode)') {
            sh '''
                export PATH=$PATH:$HOME/.local/bin
                ansible-playbook --check --connection=local --inventory=localhost, playbooks/verify.yml
            '''
        }
    }
}
