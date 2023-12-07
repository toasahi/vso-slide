---
layout: cover
highlighter: shiki
css: unocss
colorSchema: dark
title: Vault Secrets Operator
transition: fade-out
fonts:
  sans: 'Noto Sans JP'
---

<h1 flex="~ col">
<div c-yellow-300><b font-bold>Vault Secrets Operator</b></div>
<div flex="~ gap3" items-center>of <span inline-block i-logos-nuxt-icon text-1.2em mb-2/> Hashi Corp</div>
</h1>

<div abs-br mx-10 my-12 flex="~ col" text-sm text-right>
  <img src="/vault-logo.svg" h-6 alt="Vault Logo" />
  <div text-sm opacity-50>Nov. 30th 2023</div>
</div>


---
layout: intro
---
# 名前

<div class="leading-10 opacity-80">
Team: <br>
Hobby: 散歩 ラジオ ラーメン<br>
<br>
</div>

<img src="/vault-secrets-operator-arch.png" rounded-full w-35 abs-tr mt-32 mr-40 />

<div flex="~ gap2">

</div>

---
layout: 'center'
---

<h1 flex="~ col" text-5 tracking-widest>
<div mb-3>What is a <b font-bold c-yellow-300>Vault</b></div>
<div flex="~ gap3" items-center>of<span inline-block text-1.2em mb-2/> Hashi Corp ?</div>
</h1>

---
layout: 'two-cols'
---
<div flex flex-items-center h-full>
<div>
  <h1 flex="~ col" text-2 tracking-widest justify-center >
    <div mb-3>What is a<b font-bold c-yellow-300> Vault</b></div>
    <div flex="~ gap3" items-center>of<span inline-block text-1.2em mb-2/> Hashi Corp ?</div>
  </h1>

  <div>
  <h2 flex="~ col" text-6></h2>
    <p tracking-widest>Vaultで秘密を管理し、機密データを保護
    UI、CLI、HTTP API を使用して、秘密やその他の機密データを保護するためのトークン、パスワード、証明書、暗号化キーへのアクセスを保護、保存、厳重に制御します。</p>
  </div>
</div>
</div>
::right::
<div flex flex-items-center h-full pl-20>
<img src='/vault-white.svg'>
</div>

---
layout: default
---

<h1 flex="~ col">
<div c-yellow-300><b font-bold>Vault Secrets Operator</b></div>
</h1>

<div>
  <p flex="~ col" c-white>
    2023年3月にリリースされた、VaultとKubernetes間でSecretを連携させる
    Operator
  </p>
</div>
<div p-10 bg-white>
<img src='/vault-secrets-operator-arch.png' w-full>
</div>

---
clicks: 3
---

<div>
<h1 flex="~ col">
<div c-yellow-300><b font-bold>VSO Custom Resource</b></div>
</h1>

<div>

<v-clicks>

- VaultConnection
- VaultAuth
- Vault(Static,Dynamic)Secret

</v-clicks>
</div>
</div>

---
clicks: 3
---
<div>
<h1 flex="~ col">
<div><b font-bold>VaultConnection</b></div>
</h1>
<div>
<p>Vaultの接続先を定義する</p>
</div>
```yaml {*|5|7|*} {at:0}
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultConnection
metadata:
  name: vaultConnection
  namespace: vault-secrets-operator-system
spec:
  address: https://vault.com
```
</div>
---
clicks: 3
---
<div>
<h1 flex="~ col">
<div><b font-bold>VaultAuth</b></div>
</h1>
<div>
<p>Vaultに接続する際に用いるService AccountとVault上にあるRoleを指定する</p>
</div>
```yaml {*|7,8|9,10,11|*} {at:0}
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultAuth
metadata:
  name: vault-auth
  namespace: app
spec:
  method: kubernetes
  mount: k8s-dev
  kubernetes:
    role: app-role
    serviceAccount: app-sa
```
</div>
---
clicks: 4
---
<div>
<h1 flex="~ col">
<div><b font-bold>Vault(Static,Dynamic)Secret</b></div>
</h1>
<div>
<p>Vaultに登録しているSecretの場所を定義する</p>
</div>
<div grid="~ cols-2 gap-x-4 gap-y-2">

<h5>VaultStaticSecret</h5>
<h5>VaultDynamicSecret(StaticRole)</h5>

```yaml {*|7,8,9|10|11,12,13,14,15,16|*} {at:0}
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultStaticSecret
metadata:
  name: vault-static-secret
  namespace: app
spec:
  type: kv-v2
  mount: kvv2
  path: webapp/config
  refreshAfter: 30s
  destination:
    name: vault-secret
    create: true
  rolloutRestartTargets:
  - kind: Deployment
    name: app-deployment
  vaultAuthRef: vault-auth
```

```yaml {*|8,9|99|10,11,12,13,14,15|*} {at:0}
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultDynamicSecret
metadata:
  name: vault-dynamic-secret
  namespace: app
spec:
  allowStaticCreds: true
  mount: database
  path: static-creds/dev-postgres
  destination:
    create: true
    name: vault-db-secret
  rolloutRestartTargets:
  - kind: Deployment
    name: app-deployment
  vaultAuthRef: vault-auth
```
</div>
</div>
---
layout: two-cols
---
<div>
<h2 flex="~ col">
<div><b font-bold>VaultDynamicSecret(StaticRole)</b></div>
</h2>

<div pt-4>
<h4>使い道</h4>
<p>Databaseのユーザ名を固定にして、パスワードのみを動的に変更する場合に使用する</p>
</div>
</div>

<h5>Secretのローテーションのタイミング</h5>
<div>

- rotation_period
<div class='text-sm'>N秒 、N分 、N時間でローテーションのタイミングを定義</div>

```
rotation_period : 1h
```
- rotation_schedule
<div class='text-sm'>cron形式でローテーションのタイミングを定義</div>

```
0 * * * Web
```
</div> 

::right::
<div flex flex-items-center h-full flex-justify-end>

```yaml {*} 
apiVersion: secrets.hashicorp.com/v1beta1
kind: VaultDynamicSecret
metadata:
  name: vault-dynamic-secret
  namespace: app
spec:
  allowStaticCreds: true
  mount: database
  path: static-creds/dev-postgres
  destination:
    create: true
    name: vault-db-secret
  rolloutRestartTargets:
  - kind: Deployment
    name: app-deployment
  vaultAuthRef: vault-auth
```
</div>
---
layout: 'end'
---