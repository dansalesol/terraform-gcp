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

4.  Feito isso podemos criar nosso bucket no Google Storage para salvar os estados do Terraform.







