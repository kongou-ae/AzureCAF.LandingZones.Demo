# DevBox プールの作成

各開発プロジェクトで開発ボックス（仮想マシン）を作成できるようにするため、各開発プロジェクトに DevBox プールを作成します。

- DevBox プールに指定する要素は基本的に 2 つです。
  - DevCenter に定義した、VM イメージと VM SKU の組み合わせ（DevBox 定義）
  - VM の展開先となるネットワーク（アタッチドネットワーク接続）
- 上記に加えて、以下のオプションを指定します。
  - licenseType（適切なライセンスを持っていることを確認した上で "Windows_Client" を指定。必要ライセンスは[こちら](https://azure.microsoft.com/ja-jp/pricing/details/dev-box/)。）
  - 仮想マシンのローカル管理者権限を要求者に与えるか否か。

本サンプルでは、以下のような DevBox プールを作成します。

| 開発プロジェクト名 | DevBox プール名 | DevBox 定義 | アタッチドネットワーク接続 |
| --- | --- | --- | --- |
| DevProjectX | dbp-devprojectx-vs2022-8core-${TEMP_LOCATION_PREFIX} | vs2022ent-8core-32gb-256gb | an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojectx |
| DevProjectX | dbp-devprojectx-vs2022-16core-${TEMP_LOCATION_PREFIX} | vs2022ent-16core-64gb-256gb | an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojectx |
| DevProjectY | dbp-devprojecty-vs2022-8core-${TEMP_LOCATION_PREFIX} | vs2022ent-8core-32gb-256gb | an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojecty |
| DevProjectY | dbp-devprojecty-vs2022-16core-${TEMP_LOCATION_PREFIX} | vs2022ent-16core-64gb-256gb | an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojecty |

```bash

# DevCenter 構築用アカウントに切り替え
if ${FLAG_USE_SOD}; then if ${FLAG_USE_SOD_SP}; then TEMP_SP_NAME="sp_dev1_dev"; az login --service-principal --username ${SP_APP_IDS[${TEMP_SP_NAME}]} --password "${SP_PWDS[${TEMP_SP_NAME}]}" --tenant ${PRIMARY_DOMAIN_NAME} --allow-no-subscriptions; else az account clear; az login -u "user_dev1_dev@${PRIMARY_DOMAIN_NAME}" -p "${ADMIN_PASSWORD}"; fi; fi
# Dev1 サブスクリプションで作業
az account set -s "${SUBSCRIPTION_ID_DEV1}"

TEMP_LOCATION_NAME=${LOCATION_NAMES[0]}
TEMP_LOCATION_PREFIX=${LOCATION_PREFIXS[0]}
TEMP_RG_NAME="rg-devcenter-${TEMP_LOCATION_PREFIX}"
TEMP_DC_NAME="dc-devcenter-${TEMP_LOCATION_PREFIX}"

TEMP_DBP_DEFS="\
DevProjectX,dbp-devprojectx-vs2022-8core-${TEMP_LOCATION_PREFIX},vs2022ent-8core-32gb-256gb,an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojectx \
DevProjectX,dbp-devprojectx-vs2022-16core-${TEMP_LOCATION_PREFIX},vs2022ent-16core-64gb-256gb,an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojectx \
DevProjectY,dbp-devprojecty-vs2022-8core-${TEMP_LOCATION_PREFIX},vs2022ent-8core-32gb-256gb,an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojecty \
DevProjectY,dbp-devprojecty-vs2022-16core-${TEMP_LOCATION_PREFIX},vs2022ent-16core-64gb-256gb,an-vnet-devbox-${TEMP_LOCATION_PREFIX}-subnet-devprojecty \
"

for TEMP_DBP_DEF in $TEMP_DBP_DEFS; do
TEMP=(${TEMP_DBP_DEF//,/ })
TEMP_PRJ_NAME=${TEMP[0]}
TEMP_POOL_NAME=${TEMP[1]}
TEMP_DBD_NAME=${TEMP[2]}
TEMP_AN_NAME=${TEMP[3]}

# DevBox プールを対象プロジェクトに作成
# ※ 将来的に StopOnDisconnected がサポートされる予定
az rest --method PUT --uri "/subscriptions/${SUBSCRIPTION_ID_DEV1}/resourceGroups/${TEMP_RG_NAME}/providers/Microsoft.DevCenter/projects/${TEMP_PRJ_NAME}/pools/${TEMP_POOL_NAME}?api-version=2023-04-01" --body @- <<EOF
{
  "properties": {
    "devBoxDefinitionName": "${TEMP_DBD_NAME}",
    "networkConnectionName": "${TEMP_AN_NAME}",
    "licenseType": "Windows_Client",
    "localAdministrator": "Enabled"
  },
  "location": "${TEMP_LOCATION_NAME}"
}
EOF

done # TEMP_DBP_DEF

```
