#!/usr/bin/env groovy

library("govuk@broken-interpolation")

REPOSITORY = 'mapit'

DEFAULT_BRANCH = 'main'

node {

  try {
    stage('Checkout') {
      govuk.checkoutFromGitHubWithSSH(REPOSITORY)
      govuk.cleanupGit()
      govuk.mergeIntoBranch(DEFAULT_BRANCH)
    }

    stage('Installing Packages') {
      sh("rm -rf venv")
      sh("python3.6 -m venv venv")
      sh("venv/bin/python -m pip install --upgrade pip wheel setuptools")
      sh("venv/bin/python -m pip install -r requirements.txt")
    }

    stage('Tests') {
      govuk.setEnvar("GOVUK_ENV", "ci")
      sh("venv/bin/python manage.py test --noinput mapit mapit_gb")
    }

    if (env.BRANCH_NAME == DEFAULT_BRANCH) {
      stage('Push release tag') {
        govuk.pushTag(REPOSITORY, BRANCH_NAME, 'release_' + BUILD_NUMBER, DEFAULT_BRANCH)
      }

      stage('Deploy to Integration') {
        govuk.deployToIntegration(REPOSITORY, 'release_' + BUILD_NUMBER, 'deploy')
      }
    }
  } catch (e) {
    currentBuild.result = 'FAILED'
    step([$class: 'Mailer',
          notifyEveryUnstableBuild: true,
          recipients: 'govuk-ci-notifications@digital.cabinet-office.gov.uk',
          sendToIndividuals: true])
    throw e
  }

  // Wipe the workspace
  deleteDir()
}
