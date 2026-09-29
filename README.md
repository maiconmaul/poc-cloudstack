# POC Apache CloudStack com KVM e NFS

Prova de conceito acadêmica de uma nuvem privada baseada no **Apache CloudStack 4.22.1.1**, com KVM, armazenamento NFS e rede Advanced. O relatório completo está em [relatorio/RELATORIO-POC-CLOUDSTACK.md](relatorio/RELATORIO-POC-CLOUDSTACK.md).

> Status atual: infraestrutura, Management Server, host KVM, zona e storages configurados. As System VMs ainda estavam em inicialização na última evidência.

## Arquitetura do laboratório

| Componente | Configuração |
| --- | --- |
| Hipervisor externo | Oracle VirtualBox |
| Management Server | `cs-mgmt.lab.local` — `192.168.100.10/24` |
| Host KVM | `cs-kvm01.lab.local` — `192.168.100.11/24` |
| Rede de gerenciamento | VirtualBox Internal Network `cs-mgmt-net` |
| Rede pública/convidados | VirtualBox Internal Network `cs-public-net` |
| Armazenamento primário | NFS `192.168.100.10:/export/primary` |
| Armazenamento secundário | NFS `192.168.100.10:/export/secondary` |

## Evidências da execução

### 1. Host KVM e rede

KVM e aceleração por hardware foram validados no host de computação.

![KVM acceleration can be used](evidencias/03-kvm/E01-kvm-aceleracao-validada.png)

As bridges usadas pelo CloudStack foram configuradas da seguinte forma: `cloudbr0` para gerenciamento/storage e `cloudbr1` para tráfego público e de convidados.

![Bridges cloudbr0 e cloudbr1](evidencias/02-rede/E03-bridges-cloudbr0-cloudbr1.png)

### 2. NFS primário e secundário

O Management Server exporta o diretório `/export` ao host KVM com permissão de escrita para a POC.

![Export NFS configurado](evidencias/01-infraestrutura/E05-nfs-export-configurado.png)

A montagem foi validada no KVM com criação de arquivo no compartilhamento remoto.

![Montagem NFS validada](evidencias/01-infraestrutura/E06-montagem-nfs-validada.png)

### 3. Instalação do CloudStack

O repositório oficial do CloudStack 4.22 foi configurado para Ubuntu Noble, com a versão 4.22.1.1 disponível.

![Repositório CloudStack 4.22](evidencias/04-cloudstack/E07-repositorio-cloudstack-4.22.1.1.png)

O Management Server, MySQL e Java 17 foram instalados e validados.

![Pacotes do Management Server e MySQL](evidencias/04-cloudstack/E08-management-mysql-instalados.png)

Os parâmetros obrigatórios do MySQL foram aplicados: `binlog_format=ROW`, `max_connections=350` e `server_id=1`.

![Parâmetros MySQL](evidencias/04-cloudstack/E09-mysql-parametros-validados.png)

Os bancos do CloudStack foram inicializados com sucesso.

![Banco CloudStack inicializado](evidencias/04-cloudstack/E10-banco-cloudstack-inicializado.png)

### 4. Console web e template de sistema

O serviço de gerenciamento respondeu localmente na porta 8080.

![Painel CloudStack respondendo](evidencias/04-cloudstack/E12-painel-local-respondendo.png)

O console foi publicado no computador hospedeiro pelo redirecionamento NAT do VirtualBox.

![Console CloudStack no Windows](evidencias/04-cloudstack/E13-console-web-windows.png)

O login administrativo abriu o painel inicial do CloudStack.

![Login administrativo](evidencias/04-cloudstack/E14-login-administrativo.png)

O template de sistema para KVM foi instalado no armazenamento secundário.

![Template System VM instalado](evidencias/04-cloudstack/E15-template-sistema-instalado.png)

### 5. Agente KVM, zona e storages

O agente CloudStack do KVM foi configurado para se comunicar com o Management Server em `192.168.100.10:8250`.

![Agente CloudStack KVM ativo](evidencias/03-kvm/E16-agente-cloudstack-ativo.png)

A zona `POC-Zone` foi habilitada. A Secondary Storage VM iniciou o provisionamento no host KVM; esta é uma evidência de progresso, não de conclusão das System VMs.

![Zona habilitada e System VMs inicializando](evidencias/04-cloudstack/E18-zona-ativa-system-vms-inicializando.png)

## Próximas etapas

1. Confirmar a execução das System VMs.
2. Provisionar uma instância de teste.
3. Validar rede isolada, IP público e NAT.
4. Criar e testar volume e snapshot.
5. Demonstrar multitenancy e a integração Kubernetes/CSI, conforme o escopo acadêmico.

## Estrutura do repositório

```text
evidencias/       Capturas organizadas por fase da POC
relatorio/        Relatório técnico detalhado
configuracoes/    Registros de configuração relevantes
logs/             Espaço reservado para logs de validação
guia-reproducao/  Guia de importação no VirtualBox e status da POC
```

## Reprodução em outro computador

- [Guia rápido de configuração no VirtualBox](guia-reproducao/01-guia-rapido-virtualbox.md)
- [Status detalhado e próximas etapas](guia-reproducao/02-status-detalhado-poc.md)
