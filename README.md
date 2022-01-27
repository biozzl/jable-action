<h1 align="center">Jable action</h1>
<p align="center">
使用 action 和 https://github.com/hcjohn463/JableTVDownload 来上传至 release 或 webdav 中
</p>

# Config

## download-to-release

TL;DR 下载之后上传到 release 中，tag 为当前日期

* 【可选】其中主要的配置的部分是 `retry` [步骤](#重试)


## download-to-webdav

TL;DR 下载之后上传到 webdav 中，使用 tailscale 当做中转（速度较慢）（action runner 不能连接到家庭公网 ip）（懒得配置 frp）

1. 【可选】需要重新配置 `retry` [步骤](#重试)
2. 【可选】`${{ secrets.TAILSCALE_AUTHKEY }}` tailscale 的秘钥，参看 https://github.com/tailscale/github-action
3. `http://jojo-nas-one:5999` webdav 的地址
4. `${{secrets.username}}` webdav 的用户名，[放到 secrets 中](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
5. `${{secrets.password}}` webdav 的密码，[放到 secrets 中](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## tailscale-test

测试 tailscale 连接是否正常

# Details

## 重试

```yaml
- name: Retry
  if: ${{ failure() && hashFiles('err.log')}}
  shell: sh
  run: |
    if grep -q 'of \"ios\"' err.log;
    then curl --location --request POST 'https://api.github.com/repos/bxb100/jable-action/actions/workflows/18472383/dispatches' \
    --header 'Accept: application/vnd.github.v3+json' \
    --header 'Authorization: token ${{ secrets.GH_PAT }}' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "ref": "main",
        "inputs": {
            "url": "${{github.event.inputs.url}}",
            "random":"${{github.event.inputs.random}}"
        }
    }'
    fi
```

主要作用是当前面产生 `Sorry "firefox" browser was not found with a platform of "ios"` 错误的时候重新调用 workflow，如果不需要可以直接注释
### 重新配置

1. `${{ secrets.GH_PAT }}`

申请一个 token，注意需要 Update GitHub Action workflows 的权限，[放到 secrets 中](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

![image](https://user-images.githubusercontent.com/20685961/151335371-0dbc2f04-25bf-455a-b33e-4d001561798a.png)

2. `https://api.github.com/repos/bxb100/jable-action/actions/workflows/18472383/dispatches`

> API 文档：https://docs.github.com/en/rest/reference/actions#list-repository-workflows


获取你仓库下 workflow 的 id，然后替换 `用户：bxb100` `仓库名：jable-taction` 和 `workflow id：18472383`。



