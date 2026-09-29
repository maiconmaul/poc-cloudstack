# Guia rápido — Apache CloudStack no VirtualBox

Este guia restaura a POC em outro computador a partir dos ZIPs das máquinas `cs-mgmt` e `cs-kvm01`.

## Componentes

| VM | Função | IP de gerenciamento |
| --- | --- | --- |
| `cs-mgmt` | Management Server, MySQL e NFS | `192.168.100.10/24` |
| `cs-kvm01` | Host KVM e agente CloudStack | `192.168.100.11/24` |

## Requisitos do computador hospedeiro

- Oracle VirtualBox instalado.
- VT-x ou AMD-V habilitado na BIOS/UEFI.
- Virtualização aninhada disponível e habilitada no `cs-kvm01`.
- 8 GB de RAM física no mínimo; 16 GB recomendados.
- 50 GB livres em disco.

## Importar os ZIPs

1. Descompacte cada ZIP em uma pasta local.
2. Para um arquivo `.ova`, use **File → Import Appliance** no VirtualBox.
3. Para um arquivo `.vbox`, use **Machine → Add** e selecione o arquivo.
4. Importe primeiro `cs-mgmt`, depois `cs-kvm01`.
5. Mantenha as VMs desligadas durante os ajustes abaixo.

## Configurações externas no VirtualBox

### `cs-mgmt`

| Item | Configuração |
| --- | --- |
| Memória | **4096 MB** |
| Processadores | 2 vCPUs |
| Adaptador 1 | NAT, modo promíscuo Deny |
| Adaptador 2 | Internal Network `cs-mgmt-net`, modo promíscuo Deny |

No adaptador NAT, em **Advanced → Port Forwarding**, crie:

| Nome | Protocolo | IP hospedeiro | Porta hospedeiro | Porta convidado |
| --- | --- | --- | ---:| ---:|
| CloudStack UI | TCP | `127.0.0.1` | `8080` | `8080` |
| SSH opcional | TCP | `127.0.0.1` | `2222` | `22` |

### `cs-kvm01`

| Item | Configuração |
| --- | --- |
| Memória | **4096 MB recomendado**; 2048 MB é mínimo reduzido |
| Processadores | 2 vCPUs |
| Virtualização aninhada | habilitada |
| Adaptador 1 | NAT, modo promíscuo Deny |
| Adaptador 2 | Internal Network `cs-mgmt-net`, modo promíscuo Allow All |
| Adaptador 3 | Internal Network `cs-public-net`, modo promíscuo Allow All |

> O nome das redes internas precisa ser exatamente `cs-mgmt-net` e `cs-public-net`. Com 2 GB, System VMs ou a instância de teste podem ficar sem capacidade; 4 GB é a opção adequada para concluir a POC.

## Redes esperadas dentro das VMs

| VM | Interface/bridge | Finalidade | Endereço |
| --- | --- | --- | --- |
| `cs-mgmt` | `enp0s3` | NAT/internet | DHCP |
| `cs-mgmt` | `enp0s8` | gerenciamento | `192.168.100.10/24` |
| `cs-kvm01` | `enp0s3` | NAT/internet | DHCP |
| `cs-kvm01` | `cloudbr0` sobre `enp0s8` | gerenciamento e storage | `192.168.100.11/24` |
| `cs-kvm01` | `cloudbr1` sobre `enp0s9` | público e guest | sem IP |

As entradas locais de nomes devem ser:

```text
192.168.100.10 cs-mgmt.lab.local cs-mgmt
192.168.100.11 cs-kvm01.lab.local cs-kvm01
```

## Inicialização

1. Inicie `cs-mgmt` e aguarde cerca de dois minutos.
2. No Management Server, execute:

```bash
sudo systemctl start mysql nfs-kernel-server cloudstack-management
sudo systemctl status mysql nfs-kernel-server cloudstack-management --no-pager
```

3. Inicie `cs-kvm01` e execute:

```bash
sudo systemctl start libvirtd cloudstack-agent
sudo systemctl status libvirtd cloudstack-agent --no-pager
```

## Testes de conectividade

No `cs-kvm01`:

```bash
ping -c 3 192.168.100.10
timeout 3 bash -c '</dev/tcp/192.168.100.10/8250' && echo '8250 aberta' || echo '8250 falhou'
sudo systemctl is-active cloudstack-agent
sudo tail -n 30 /var/log/cloudstack/agent/agent.log
```

O esperado é: ping sem perda, `8250 aberta`, agente `active` e nenhuma tentativa de conexão para `10.0.2.15:8250`. O endpoint correto do agente é `192.168.100.10:8250`.

No `cs-mgmt`:

```bash
sudo systemctl is-active cloudstack-management
sudo systemctl is-active mysql
sudo systemctl is-active nfs-kernel-server
```

Os três serviços devem retornar `active`.

## Painel web

Abra no computador hospedeiro:

```text
http://127.0.0.1:8080/client/
```

| Campo | Valor |
| --- | --- |
| Usuário | `admin` |
| Senha padrão CloudStack | `password` |
| Domínio | deixar em branco |

> Troque a senha padrão se o laboratório for exposto fora do computador local.

## Validação no painel

1. Em **Infrastructure → Zones**, confirme `POC-Zone` como `Enabled`.
2. Confirme o host `cs-kvm01.lab.local` como `Up`.
3. Confirme `Primary-NFS` e `Secondary-NFS` como `Up`.
4. Em **System VMs**, aguarde a Secondary Storage VM e a Console Proxy VM ficarem `Running`.

## Solução rápida de problemas

| Sintoma | Ação |
| --- | --- |
| Painel não abre | Revise o port forwarding e o serviço `cloudstack-management`. |
| Agente não conecta | Confira `host=192.168.100.10` e `port=8250` em `/etc/cloudstack/agent/agent.properties`. |
| Falta capacidade | Aumente `cs-kvm01` para 4 GB e reinicie a VM. |
| NFS falha | Valide `nfs-kernel-server`, o export `/export` e o ping entre os dois IPs. |
| System VMs não iniciam | Confirme zona habilitada, storages `Up` e RAM disponível no KVM. |

## Links

- [Status detalhado da POC](02-status-detalhado-poc.md)
- [Relatório de execução](../relatorio/RELATORIO-POC-CLOUDSTACK.md)
- [Evidências](../evidencias/)
