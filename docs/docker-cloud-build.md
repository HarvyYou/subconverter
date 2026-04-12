# 云端构建 Docker 镜像（GitHub Actions）

本仓库已包含工作流 **[`.github/workflows/docker.yml`](../.github/workflows/docker.yml)**：在 GitHub 上推送代码后，由 **GitHub Actions** 在云端编译并推送镜像到 **Docker Hub**。

镜像名格式为：**`你的 GitHub 用户名/subconverter`**（例如 `harvyyou/subconverter`），与上游作者镜像相互独立。

---

## 你需要准备的内容

1. **代码在 GitHub 上**  
   将 fork 后的仓库推送到你自己的账号下（默认分支名为 `master` 时与当前工作流一致；若你使用 `main`，需改工作流里的 `branches`）。

2. **Docker Hub 账号**  
   在 [Docker Hub](https://hub.docker.com) 注册；镜像仓库名建议为 **`subconverter`**（完整路径：`用户名/subconverter`）。首次推送前可在网页上创建该仓库，或让首次 `push` 时按提示创建。

3. **在 GitHub 仓库里配置密钥**  
   打开：**Settings → Secrets and variables → Actions → New repository secret**，添加：

   | Name | 值 |
   |------|-----|
   | `DOCKER_USERNAME` | 你的 Docker Hub 用户名（需与镜像前缀一致，例如 `harvyyou`） |
   | `DOCKER_PASSWORD` | Docker Hub **密码**或 **[Access Token](https://hub.docker.com/settings/security)**（推荐 Token） |

   保存后，工作流里的 `docker/login-action` 会用它们登录并推送。

---

## 触发方式

工作流在以下情况会运行：

- 推送到分支 **`master`**
- 推送任意 **git tag**（常用于发版，如 `v1.0.0`）

推送完成后，在仓库 **Actions** 页可查看进度；成功后在 Docker Hub 可看到新镜像与标签（如 `latest`、semver 标签等）。

### 仅通过 `master` 做云端构建时，需要做什么？

当前工作流配置为：**只有 `master` 上出现新的 push 才会构建镜像**（外加打 tag 时）。

因此常见流程是：

1. 在其它分支开发、提交，并推送到 GitHub。  
2. 通过 **Pull Request** 把改动 **合并进 `master`**（或在网页上直接 Merge）。  
3. 合并成功后，GitHub 会向 **`master` 产生一次 push**，这会自动触发 **Actions**，**无需**再手动点「运行工作流」（除非你希望额外再跑一遍）。

若你在本机直接执行 `git push origin master`，同样会触发。

**注意：** 若你仓库的默认分支是 **`main`** 而不是 `master`，需要么把默认开发流程改到 `master`，要么把 [`.github/workflows/docker.yml`](../.github/workflows/docker.yml) 里的 `branches: [ master ]` 改成 `[ main ]`，否则推 `main` 不会触发本工作流。

---

## 工作流行为说明（与本仓库 Dockerfile 一致）

- **构建上下文**为仓库**根目录**，Dockerfile 为 **`scripts/Dockerfile`**（与本地 `docker build -f scripts/Dockerfile .` 一致）。
- **`REGISTRY_IMAGE`** 使用 **`${{ github.repository_owner }}/subconverter`**，因此不同 fork 会自动推到各自用户名下，无需改 YAML 里的固定作者名。
- 多架构构建（amd64 / 386 / arm 等）与清单合并逻辑仍由原工作流保留；若你只需要 `linux/amd64`，可自行简化 `matrix`。

若 Docker Hub 要求**全小写**仓库名，而你的 GitHub 用户名含大写字母，推送可能失败；此时可把镜像名改为小写（需同时调整工作流里的镜像命名逻辑或使用 `docker/metadata-action` 的规范化选项）。

---

## 其它云端方式（简要）

| 方式 | 说明 |
|------|------|
| **GitLab CI** | 在 `.gitlab-ci.yml` 里使用 `docker build` + `docker push`，并配置 Registry 变量。 |
| **云厂商构建** | 如 AWS CodeBuild、GCP Cloud Build、阿里云 ACR 企业版等，上传源码或连接 Git，按向导配置 Dockerfile 路径与镜像地址。 |

核心都是：**在能访问你代码的环境里执行与本地相同的 `docker build`（根目录上下文 + `scripts/Dockerfile`），再 `docker push` 到目标 Registry。**
