# Relatório de execução - POC Apache CloudStack

## Objetivo

Implementar e validar uma nuvem privada de laboratório com Apache CloudStack 4.22.1.1, KVM, NFS e rede Advanced, conforme o documento-base da disciplina.

## Ambiente de laboratório

| Componente | Configuração atual |
| --- | --- |
| Hipervisor externo | Oracle VirtualBox no Windows |
| Sistema operacional dos nós | Ubuntu Server 24.04.5 LTS amd64 |
| Management Server | `cs-mgmt.lab.local` - `192.168.100.10/24` |
| Host KVM | `cs-kvm01.lab.local` - `192.168.100.11/24` |
| Rede de gerenciamento | Rede Interna VirtualBox `cs-mgmt-net` |
| Rede pública/guest | Rede Interna VirtualBox `cs-public-net` (em preparação) |

## Registro de execução

### E01 - Virtualização KVM validada

- **Nó:** `cs-kvm01.lab.local`
- **Procedimento:** instalação de QEMU/KVM, libvirt e ferramentas de validação.
- **Comandos executados:**

  ```bash
  sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils cpu-checker
  sudo systemctl enable --now libvirtd
  sudo kvm-ok
  sudo virsh list --all
  ```

- **Resultado observado:** `/dev/kvm exists` e `KVM acceleration can be used`; não havia VMs libvirt pré-existentes.
- **Status:** aprovado.
- **Evidência:** salvar a captura em `evidencias/03-kvm/E01-kvm-aceleracao-validada.png`.

### E02 - Comunicação da rede de gerenciamento

- **Nós:** `cs-mgmt.lab.local` e `cs-kvm01.lab.local`.
- **Configuração:** endereços estáticos `192.168.100.10/24` e `192.168.100.11/24`; resolução de nomes definida em `/etc/hosts` nos dois nós.
- **Procedimento:** ping de `cs-kvm01` para `cs-mgmt.lab.local`.
- **Resultado observado:** comunicação validada pelo operador.
- **Status:** aprovado.
- **Evidência pendente:** capturar a saída de `ping -c 3 cs-mgmt.lab.local` e salvar como `evidencias/02-rede/E02-ping-gerenciamento.png`.

### E03 - Pré-requisitos do Management Server

- **Nó:** `cs-mgmt.lab.local`.
- **Resultado observado:** FQDN resolvido, SSH ativo, Chrony ativo e endereço de gerenciamento configurado.
- **Status:** aprovado.
- **Evidência pendente:** capturar `hostname -f`, `systemctl is-active ssh chrony` e `ip -br a`.

### E04 - Bridges de rede do host KVM

- **Nó:** `cs-kvm01.lab.local`.
- **Configuração:** `cloudbr0` criada sobre `enp0s8` para tráfego de gerenciamento/storage; `cloudbr1` criada sobre `enp0s9` para tráfego público/guest.
- **Procedimento:** aplicação da configuração Netplan e validação com `ip -br a` e `bridge link`.
- **Resultado observado:** `cloudbr0` ativa com o endereço `192.168.100.11/24`; `cloudbr1` ativa; `enp0s8` e `enp0s9` associados às bridges corretas.
- **Status:** aprovado.
- **Evidência:** `evidencias/02-rede/E03-bridges-cloudbr0-cloudbr1.png`.

### E05 - Java e serviço NFS do Management Server

- **Nó:** `cs-mgmt.lab.local`.
- **Procedimento:** instalação de OpenJDK 17 e `nfs-kernel-server`.
- **Resultado observado:** OpenJDK `17.0.20.1` instalado; serviço `nfs-server` no estado `active`.
- **Status:** aprovado.
- **Evidência recomendada:** `evidencias/01-infraestrutura/E04-java-nfs-ativos.png`.

### E06 - Export NFS configurado

- **Nó servidor:** `cs-mgmt.lab.local`.
- **Export:** `/export` autorizado exclusivamente para `192.168.100.11` com escrita e `no_root_squash`.
- **Procedimento:** criação de `/export/primary` e `/export/secondary`; recarga com `exportfs -ra`; inspeção com `exportfs -v`.
- **Resultado observado:** export ativo com permissões `rw`, `async` e `no_root_squash`.
- **Status:** aprovado.
- **Evidência:** `evidencias/01-infraestrutura/E05-nfs-export-configurado.png`.

### E07 - Montagem NFS validada pelo host KVM

- **Origem:** `cs-kvm01.lab.local`.
- **Destino:** `192.168.100.10:/export`.
- **Procedimento:** montagem NFSv4 em `/mnt/poc-nfs` e criação de `validacao-kvm.txt` com privilégios root.
- **Resultado observado:** montagem concluída e arquivo criado no storage remoto; confirma comunicação, permissões de escrita e `no_root_squash`.
- **Status:** aprovado.
- **Evidência:** `evidencias/01-infraestrutura/E06-montagem-nfs-validada.png`.

### E08 - Repositório CloudStack 4.22 validado

- **Nó:** `cs-mgmt.lab.local`.
- **Configuração:** repositório APT `https://download.cloudstack.org/ubuntu noble 4.22` e chave pública do projeto.
- **Procedimento:** `apt update` e `apt-cache policy cloudstack-management`.
- **Resultado observado:** versão candidata `4.22.1.1` disponível.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E07-repositorio-cloudstack-4.22.1.1.png`.

### E09 - Pacotes do Management Server e MySQL instalados

- **Nó:** `cs-mgmt.lab.local`.
- **Pacotes:** `cloudstack-management 4.22.1.1`, `mysql-server 8.0.46` e OpenJDK 17.
- **Procedimento:** instalação via APT e verificação por `dpkg -l` e `java -version`.
- **Resultado observado:** versões instaladas conforme o repositório CloudStack 4.22 e requisitos de Java.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E08-management-mysql-instalados.png`.

### E10 - Parâmetros do MySQL validados

- **Nó:** `cs-mgmt.lab.local`.
- **Configuração:** arquivo `/etc/mysql/conf.d/cloudstack.cnf` com rollback InnoDB, binlog em formato `ROW`, `server_id=1` e 350 conexões máximas.
- **Resultado observado:** serviço MySQL ativo; `binlog_format=ROW`, `max_connections=350` e `server_id=1` confirmados por consulta SQL.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E09-mysql-parametros-validados.png`.

### E11 - Bancos de dados do CloudStack inicializados

- **Nó:** `cs-mgmt.lab.local`.
- **Procedimento:** `sudo cloudstack-setup-databases cloud:csLab2026@localhost --deploy-as=root`.
- **Resultado observado:** criação e inicialização concluídas com sucesso. O instalador identificou `192.168.100.10` como IP do nó de gerenciamento e exibiu a confirmação de banco inicializado.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E10-banco-cloudstack-inicializado.png`.

### E12 - Management Server configurado e ativo

- **Nó:** `cs-mgmt.lab.local`.
- **Procedimento:** `sudo cloudstack-setup-management` e `sudo systemctl is-active cloudstack-management`.
- **Resultado observado:** configuração concluída e serviço retornando `active`.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E11-management-server-ativo.png`.

### E13 - Painel web do CloudStack respondendo localmente

- **Nó:** `cs-mgmt.lab.local`.
- **Validações:** memória da VM ampliada para 4 GB, serviço `cloudstack-management` ativo e requisição `curl -I http://localhost:8080/client/`.
- **Resultado observado:** resposta `HTTP/1.1 302 Found`, redirecionando para `/client/`.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E12-painel-local-respondendo.png`.

### E14 - Console web acessível pelo Windows

- **Acesso:** redirecionamento NAT do VirtualBox para a porta TCP 8080 do nó `cs-mgmt`.
- **URL:** `http://127.0.0.1:8080/client/`.
- **Resultado observado:** tela de autenticação do Apache CloudStack exibida no navegador do Windows.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E13-console-web-windows.png`.

### E15 - Autenticação administrativa no CloudStack

- **Acesso:** console web local pelo Windows.
- **Resultado observado:** sessão administrativa aberta e assistente inicial do CloudStack exibido, ainda sem zona criada.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E14-login-administrativo.png`.

### E16 - Template de sistema KVM instalado no armazenamento secundário

- **Nó:** `cs-mgmt.lab.local`.
- **Procedimento:** instalação do template SystemVM CloudStack 4.22 no diretório NFS `/export/secondary`.
- **Resultado observado:** script finalizado com confirmação de instalação bem-sucedida.
- **Status:** aprovado.
- **Evidência:** `evidencias/04-cloudstack/E15-template-sistema-instalado.png`.

### E17 - Agente CloudStack KVM ativo

- **Nó:** `cs-kvm01.lab.local`.
- **Configuração:** agente apontado para o Management Server `192.168.100.10`, GUID `cs-kvm01.lab.local`, `cloudbr0` para rede privada e `cloudbr1` para redes pública e de convidados.
- **Validação:** `sudo systemctl is-active cloudstack-agent`.
- **Resultado observado:** retorno `active`.
- **Status:** aprovado.
- **Evidência:** `evidencias/03-kvm/E16-agente-cloudstack-ativo.png`.

### E18 - Zona habilitada e System VMs em inicialização

- **Zona:** `POC-Zone`.
- **Resultado observado:** estado de alocação alterado para `Enabled`; a Secondary Storage VM (`s-2-VM`) iniciou o provisionamento no host `cs-kvm01.lab.local`. A Console Proxy VM (`v-1-VM`) ainda estava parada no instante da captura.
- **Status:** em andamento — não constitui validação final das System VMs.
- **Evidência:** `evidencias/04-cloudstack/E18-zona-ativa-system-vms-inicializando.png`.

## Próximo marco

Confirmar as System VMs em execução e validar a capacidade de memória do host KVM antes do provisionamento da primeira instância.

## Critérios de aceitação restantes

- Confirmação das System VMs em execução.
- Provisionamento de VM, rede/NAT, volume e snapshot.
- Multitenancy, Kubernetes/CKS, CSI e teste de resiliência.
