# sops-flux-playground

For this time, I decided to use kind as my kubernetes cluster. To start I have to install kind:

#### Install Kind

```bash
brew install kind
```

#### Create Cluster

```bash
kind create cluster
```

#### Export github user & token

```bash
export GITHUB_USER=[github_user]
export GITHUB_TOKEN=[github_token]
```

#### CD into the github config file
```bash
cd ~/.config/gh
```

#### Create a folder for the path

```bash
mkdir flex-boostrap-demos.pat
```

#### Create repo with Flux command

```bash
flux bootstrap github --hostname=github.com --owner=[username] --repository=sops-flux-playground --branch=main --path=clusters/kind0 --personal --private=false
```

#### Clone the repo that you just created
```bash
git clone https://github.com/[username]/sops-flux-playground.git
```

#### Open vscode

```bash
cd sops-flux-playground
code .
```

#### Modify or Change the following file
```yaml
#gotk-sync.yaml
#In the Spec, you will add the following:
spec:
	decryption:
		provider: sops
		secretRef:
			name: sops-gpg
```

#### Install gnupg sops
```bash
brew install gnupg sops
```

#### Run a GPG command / create the key
```bash
gpg --batch --full-generate-key <<EOF                                 
%no-protection
Key-Type: 1
Key-Length: 4096
Subkey-Type: 1
Subkey-Length: 4096
Expire-Date: 0
Name-Email: kind0@stealthybox.com
Name-Real: kind0
EOF
```

#### List your secrets
```bash
gpg --list-secret-keys
```

##### This should look like this:
```bash

sec   rsa4096 2023-03-09 [SCEA]
      ABB063E31C65B14C81B607021CC77C4A64215338
uid           [ultimate] kind0 <kind0@stealthybox.com>
ssb   rsa4096 2023-03-09 [SEA]

```

#### Export Secret and create a secret inside of the kubernetes cluster

```bash
gpg --export-secret-keys --armor ABB063E31C65B14C81B607021CC77C4A64215338 | kubectl create secret generic --namespace flux-system --from-file=sops.asc=/dev/stdin sops-gpg
```

#### Export public key and move it to your repo

```bash
gpg --export --armor ABB063E31C65B14C81B607021CC77C4A64215338 > clusters/kind0/flux-system/sops.pub.asc
```

##### Look
Pay attention to the path where you are located on the terminal

#### Add/Commit/Push to Github
```bash

git add .
git commit -m "Adding secrets into my repo"
git push
```

##### Attention
We recommend to create a secret for each environment you have

#### Time to reconcile your source

```bash
flux reconcile source git flux-system
```

#### Confirm all the changes that you did

```bash
kubectl describe kustomizations.kustomize.toolkit.fluxcd.io -A
```

#### Confirm the secrets
```bash

kubectl describe kustomizations.kustomize.toolkit.fluxcd.io -A | grep sops
```

By now, you should see that your repo is up-to-date and ready to use the secrets that you created.

