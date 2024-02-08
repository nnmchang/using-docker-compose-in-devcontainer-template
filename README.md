# [nnmchang/using-docker-compose-in-devcontainer-template](https://github.com/nnmchang/using-docker-compose-in-devcontainer-template)

## 概要

devcontainer内でdockerを使用するテンプレートです。  
docker-in-docker、docker-outside-of-docker、及びdevcontainerを使用していない環境で共通のdocker-composeファイルを使用することが出来ます。

## 解説

devcontainerは開発環境を統一するための素晴らしいツールですが、devcontainer内でdockerを使用する手法であるdocker-outside-of-dockerにはvolumeのマウント設定に難があります。

docker-outside-of-dockerを使用する場合、ホストの Docker を使用する関係上ホストマシンから見たパスを指定する必要がありますが、  
docker-outside-of-dockerではホストマシンとは別の環境内で`docker compose`を使用する関係上、ホストマシンのdocker daemonにはdevcontainer内でのファイルパスが通知されてしまうため、マウント対象のパスが存在せずに実行できないという問題が発生します。

これを解決するため、当リポジトリのdocker-outside-of-dockerの[devcontainer.json](https://github.com/nnmchang/using-docker-compose-in-devcontainer-template/blob/main/.devcontainer/docker-outside-of-docker/devcontainer.json)ではdevcontainerの実行時にホストマシンでの実行パスを環境変数として`$LOCAL_WORKSPACES_FOLDER`を設定しています。

この`$LOCAL_WORKSPACES_FOLDER`を[docker-compose.yml](https://github.com/nnmchang/using-docker-compose-in-devcontainer-template/blob/main/docker-compose.yml)内で`${LOCAL_WORKSPACES_FOLDER:-.}`として使用することで、  
`$LOCAL_WORKSPACES_FOLDER`が設定されている場合、つまりdocker-outside-of-dockerで実行されている場合はホストマシンでのパスとして解釈され、  
`$LOCAL_WORKSPACES_FOLDER`が未設定である場合、つまりdocker-in-docker、又はdevcontainerを使用しない環境の場合は`.`（カレントディレクトリ）として解釈される為、共通して利用可能なdocker-compose定義となります。

図解すると以下のようなイメージです。

- ホストマシンで実行するワークスペースパス
  `/home/dev/workspaces/project`

- devcontainer内でのワークスペースパス
  `/workspaces/project`

- マウントするパス
  `/public`

| 実行環境                 | $LOCAL_WORKSPACES_FOLDER     | dockerから見たパスの解釈                        |
| ------------------------ | ---------------------------- | ----------------------------------------------- |
| ホストで実行             | 未設定                       | ./public → /home/dev/workspaces/project/public |
| docker-inside-of-docker  | 未設定                       | ./public → /workspaces/project/public          |
| docker-outside-of-docker | /home/dev/workspaces/project | /home/dev/workspaces/project/public             |
