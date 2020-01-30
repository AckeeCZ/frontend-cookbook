PipelineReact {
  slackChannel = '#ci-frontend-cookbook'
  buildCommand = [
    master: 'npm install && npm run gitbook:install && npm run build',
    stage: 'npm install && npm run gitbook:install && npm run build',
    development: 'npm install && npm run gitbook:install && npm run build',
  ]
  baseURL = 'frontend-cookbook'
  buildDir = '_book'
  bucketURL = [
    master: "gs://${baseURL}.ack.ee/",
    development: "gs://${baseURL}-development.ack.ee/",
  ]
  nodeEnv = '-e NODE_PATH=./app:./config'
  nodeImage = 'node:8'
  excludeDir = '.git'
}
