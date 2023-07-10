# Desafio Pipeline CICD Cloud Build + Terraform.

1. Utilize o git https://github.com/digitalinnovationone/terraform-gcp/tree/main/terraform-exemplo2 para configurar a trigger do Cloud Build.

2. Edite o script para trocar o nome da máquina para cloudbbuildterraform.

3. Salve os arquivos do Estado no Google Cloud Storage.

Submeta o print de cada etapa de configuração do Pipeline .

-----------------------------------Resolução-------------------------------------------
1.  O primeiro passo é clonar o repositório do endereço acima.

![Captura de tela de 2023-07-10 14-29-25](https://github.com/dansalesol/terraform-gcp/assets/58992916/d15a6fab-84bf-4510-9051-cd6f4bd0e4af)

2.  Agora vou deletar a pasta oculta ".git" para que não ficar vínculado ao GitHub (vamos utilizar o Google Source Repositories como repositório).

![Captura de tela de 2023-07-10 14-32-04](https://github.com/dansalesol/terraform-gcp/assets/58992916/2ee2854d-8524-4232-86ae-6c4adf105826)

3.  O próximo passo é alterar o arquivo "main.tf" para o nosso projeto (sb-devops-iac):

```
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }

   backend "gcs" {
    bucket  = "sbterraform"
    prefix  = "terraform/state"
  }
  
}

provider "google" {
  project = "sb-devops-iac"
  region  = "us-central1"
  zone    = "us-central1-c"
}

resource "google_compute_network" "vpc_network" {
  name = "${var.network_name}"
}

resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags = ["prod"]

  labels = {
    centro_custo = "${var.centro_custo_rh}"
  }

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-11"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}
```

4.  Feito isso, podemos criar nosso bucket no Google Storage para salvar os estados do Terraform. O nome do bucket será "sbterraform".

![Captura de tela de 2023-07-10 15-07-26](https://github.com/dansalesol/terraform-gcp/assets/58992916/a55461e7-fd60-4079-882f-83560024985f)

![Captura de tela de 2023-07-10 15-08-28](https://github.com/dansalesol/terraform-gcp/assets/58992916/5c725149-4e3e-4330-9394-610761bf9b42)

5.  Agora vamos no Google Source Repositories criar um novo repositório para os arquivos do projeto. O nome do repositório será "terraform-cloudbuild".

![Captura de tela de 2023-07-10 15-12-15](https://github.com/dansalesol/terraform-gcp/assets/58992916/88910077-4723-447d-945b-77280352cb25)

![Captura de tela de 2023-07-10 15-12-32](https://github.com/dansalesol/terraform-gcp/assets/58992916/302e386a-812c-4e52-945a-78982ff19413)

![Captura de tela de 2023-07-10 15-15-39](https://github.com/dansalesol/terraform-gcp/assets/58992916/05227048-7e26-4089-9e50-200562056a4e)

6.  Vamos escolhar "enviar código a partir de um repositório Git local".

![Captura de tela de 2023-07-10 15-21-22](https://github.com/dansalesol/terraform-gcp/assets/58992916/4f5884bf-980d-431a-b991-39c45024a099)

![Captura de tela de 2023-07-10 15-25-41](https://github.com/dansalesol/terraform-gcp/assets/58992916/eb452631-bfbb-470a-bd80-618c21915a4b)

Precisamos inicializar nosso Git! Vamos fazer isso de dentro da pasta local do projeto.

![Captura de tela de 2023-07-10 15-30-46](https://github.com/dansalesol/terraform-gcp/assets/58992916/181822aa-9704-42eb-852f-223e489146e4)

Agora vamos adicionar todos os arquivos com:

![Captura de tela de 2023-07-10 15-32-05](https://github.com/dansalesol/terraform-gcp/assets/58992916/8c62b4d2-a0f7-421e-afa4-a2a23af46e05)

Vamos dar um "commit" para versionar os arquivos.

![Captura de tela de 2023-07-10 15-34-08](https://github.com/dansalesol/terraform-gcp/assets/58992916/5d7c087d-d8e5-409f-94dc-49c10cec5c5f)

Vamos ainda adicionar o repositório com:

![Captura de tela de 2023-07-10 15-38-23](https://github.com/dansalesol/terraform-gcp/assets/58992916/69303074-6ebf-4b5f-9366-52c96ec4664f)

Agora podemos dar "push" para salvar no Google Source Repositories.

![Captura de tela de 2023-07-10 15-46-53](https://github.com/dansalesol/terraform-gcp/assets/58992916/399bd55c-9a24-401c-87a2-f559f2275121)

Agora os arquivos estão salvos remotamente.

![Captura de tela de 2023-07-10 15-48-03](https://github.com/dansalesol/terraform-gcp/assets/58992916/ad183782-693a-4dfa-9b60-4ca56dd0929f)

7.  Vamos automatizar o processo com Cloud Build. Neste passo vamos primeiro encontrar o repositório clicando em "gerenciar repositórios" para verificar se o Cloud Build reconheceu o repositório..

![Captura de tela de 2023-07-10 15-53-27](https://github.com/dansalesol/terraform-gcp/assets/58992916/76e43771-3d26-40e2-8708-0392b76d6600)

Repositório encontrado! Vamos agora "criar o gatilho". Usaremos o mesmo nome dado para o repositório e iremos selecionar "enviar para uma ramificação" assim quando for feito um commit novo na brunch master ele irá executar os passos do pipeline. Vamos deixar também o arquivo de instrução como "yaml" com o nome "cloudbuild.yaml". Esse arquivo chamará o Terraform (main.tf) para executar. A conta de serviço que irá acionar será a padrão.

![Captura de tela de 2023-07-10 16-05-40](https://github.com/dansalesol/terraform-gcp/assets/58992916/f99105ab-d68b-4283-9946-3fa33c14cc01)

![Captura de tela de 2023-07-10 16-08-58](https://github.com/dansalesol/terraform-gcp/assets/58992916/0cdcbba1-9da6-47be-ad2e-4b6a2f2fe96a)

![Captura de tela de 2023-07-10 16-07-16](https://github.com/dansalesol/terraform-gcp/assets/58992916/7926ee9b-818f-4a10-86bc-9605566599de)

![Captura de tela de 2023-07-10 16-12-58](https://github.com/dansalesol/terraform-gcp/assets/58992916/97c169da-0539-44da-bdc9-c65391b82607)

Trigger criada!

8. Vamos inicializar o Terraform no diretório!

![Captura de tela de 2023-07-10 16-39-53](https://github.com/dansalesol/terraform-gcp/assets/58992916/4aaa0b5a-d740-4e02-937d-5166f34ef76c)

9. Faremos um commit para que a trigger seja disparada. Para tal, faremos uma pequena alteração como adição de um comentário.

![Captura de tela de 2023-07-10 16-20-29](https://github.com/dansalesol/terraform-gcp/assets/58992916/1124b704-db4a-4652-98fb-b695598b0eae)























