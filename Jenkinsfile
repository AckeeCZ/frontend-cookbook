PipelineReact {
  slackChannel = '#ci-frontend-cookbook'
  buildCommand = [
    master: '',
    stage: '',
    development: '',
    "feature/docsify-migration": '',
  ]
  baseURL = 'frontend-cookbook'
  buildDir = './'
  bucketURL = [
    master: "gs://${baseURL}.ack.ee/",
    development: "gs://${baseURL}-development.ack.ee/",
    "feature/docsify-migration": "gs://${baseURL}-docsify.ack.ee/",
  ]
  nodeEnv = '-e NODE_PATH=./app:./config'
  nodeImage = 'node:8'
  excludeDir = '.git'
}
