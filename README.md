# 概要


### GCPプロジェクトを作成する
- GCPプロジェクトIDを取得する


### 定義
- export PROJECT_ID=cloudrun1-381511 // 先ほど取得したものに変える
- export SERVICE_ACCOUNT=github-actions // こちらのテンプレでは固定
- export POOL_NAME=github-actions-pool // 適宜変更
- export PROVIDER_NAME=github-actions-provider // 適宜変更
- export GITHUB_REPO=inoue2002/ts-linebot-cloudrun-template // CDするリポジトリ

#### プロジェクトの設定
gcloud config set project ${PROJECT_ID}

#### APIの有効化
gcloud services enable \
		iamcredentials.googleapis.com \
		run.googleapis.com
#### サービスアカウントの作成（コピペ）
 gcloud iam service-accounts create ${SERVICE_ACCOUNT}
#### POOLの作成（コピペ）
gcloud beta iam workload-identity-pools create "${POOL_NAME}" \
  --location="global" 
#### プロバイダーの作成（コピペ）
 gcloud beta iam workload-identity-pools providers create-oidc "${PROVIDER_NAME}" \
  --location="global" \
  --workload-identity-pool="${POOL_NAME}" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository,attribute.actor=assertion.actor,attribute.aud=assertion.aud" \
  --issuer-uri="https://token.actions.githubusercontent.com"

#### サービスアカウントに権限借用の設定（コピペ）
  POOL_ID=$(gcloud beta iam workload-identity-pools describe "${POOL_NAME}" \
    --location="global" \
    --format="value(name)")
gcloud beta iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/${POOL_ID}/attribute.repository/${GITHUB_REPO}" 

#### サービスアカウントのロール設定
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member="serviceAccount:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
--role="roles/run.admin"

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member="serviceAccount:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
--role="roles/storage.admin"

gcloud projects add-iam-policy-binding ${PROJECT_ID} \
--member="serviceAccount:${SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com" \
--role="roles/iam.serviceAccountUser"