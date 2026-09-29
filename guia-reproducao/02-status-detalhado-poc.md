# Status detalhado da POC Apache CloudStack

**Atualização:** 29 de setembro de 2026  
**Conclusão estimada:** **50%**

O percentual considera a POC completa: infraestrutura, nuvem funcional, instância de usuário, rede/NAT, armazenamento, multitenancy, Kubernetes/CSI e resiliência. A base foi instalada, mas os testes práticos de consumo da nuvem ainda precisam ser concluídos.

## Resumo por etapa

| Etapa | Situação | Progresso |
| --- | --- | ---: |
| Infraestrutura VirtualBox e redes | Concluída | 100% |
| KVM, bridges e agente | Concluída | 100% |
| Management Server, MySQL e NFS | Concluída | 100% |
| Zona, Pod, Cluster e storages | Concluída | 100% |
| System VMs | Em andamento | 50% |
| Instância, rede isolada e NAT | Pendente | 0% |
| Volume e snapshot | Pendente | 0% |
| Multitenancy | Pendente | 0% |
| Kubernetes e CSI | Pendente | 0% |
| Resiliência e relatório final | Pendente | 0% |

## O que foi feito

### Infraestrutura e virtualização

- Duas VMs Ubuntu 24.04.5 LTS definidas: `cs-mgmt` e `cs-kvm01`.
- Virtualização KVM validada no host de computação.
- Bridges `cloudbr0` e `cloudbr1` configuradas.
- Rede de gerenciamento configurada em `192.168.100.0/24`.
- DNS local via `/etc/hosts` para `cs-mgmt.lab.local` e `cs-kvm01.lab.local`.

### Management Server e serviços

- OpenJDK 17 instalado.
- MySQL configurado com `binlog_format=ROW`, `server_id=1` e `max_connections=350`.
- Banco CloudStack criado por `cloudstack-setup-databases`.
- Serviço `cloudstack-management` ativo e console disponível em `127.0.0.1:8080`.
- NFS configurado no Management Server.

### Armazenamento

- Export NFS `/export` liberado ao KVM para a POC.
- Diretórios `/export/primary` e `/export/secondary` criados.
- Montagem e escrita NFS testadas no KVM.
- Template SystemVM KVM instalado no Secondary Storage.
- `Primary-NFS` e `Secondary-NFS` cadastrados na `POC-Zone` e relatados como `Up`.

### CloudStack

- CloudStack 4.22.1.1 instalado pelo repositório oficial.
- Agente do KVM configurado para `192.168.100.10:8250` e registrado.
- `POC-Zone`, `POC-Pod` e `POC-KVM-Cluster` criados.
- Host `cs-kvm01.lab.local` adicionado ao cluster.
- Faixa pública simulada `192.168.200.10` a `192.168.200.30` cadastrada.
- Zona habilitada e provisionamento das System VMs iniciado.

## Estado atual e atenção necessária

A última captura mostrou:

- `POC-Zone` como `Enabled`.
- Secondary Storage VM em `Starting` no host KVM.
- Console Proxy VM em `Stopped`.
- O painel reportando somente **0,92 GB** no host KVM.

Antes de criar instâncias de usuário, confirme a memória efetiva da VM `cs-kvm01` no VirtualBox. Para a POC funcionar com System VMs e uma instância pequena, use pelo menos 2 GB; o recomendado é 4 GB. Depois de reiniciar o KVM, o painel deve apresentar aproximadamente 1,92 GB ou 3,92 GB de memória, de acordo com o valor escolhido.

## Próximas ações detalhadas

### 1. Concluir System VMs

1. Confirmar a memória configurada no VirtualBox.
2. Reiniciar somente `cs-kvm01`, se houver alteração de RAM.
3. Confirmar que `cloudstack-agent` voltou a `active`.
4. Aguardar Secondary Storage VM e Console Proxy VM ficarem `Running`.
5. Capturar evidência definitiva das System VMs.

### 2. Provisionar instância e rede

1. Registrar uma ISO ou template de usuário.
2. Criar uma service offering pequena.
3. Criar rede isolada de convidados.
4. Criar uma instância de teste.
5. Validar DHCP, console e conectividade.
6. Adquirir IP público e configurar NAT estático ou port forwarding.

### 3. Armazenamento de instância

1. Criar volume de dados adicional.
2. Anexar à instância, formatar e escrever dados.
3. Criar snapshot e validar estado concluído.
4. Demonstrar restauração ou uso do snapshot, se houver tempo.

### 4. Multitenancy e Kubernetes

1. Criar conta, domínio ou projeto adicional.
2. Demonstrar isolamento de recursos e permissões.
3. Criar/integrar Kubernetes conforme o roteiro da disciplina.
4. Validar CSI com PersistentVolumeClaim, se o cluster estiver disponível.

### 5. Resiliência e documentação

1. Executar teste controlado de indisponibilidade de host ou serviço.
2. Registrar o comportamento esperado e a recuperação.
3. Atualizar relatório e README com resultados finais.

## Critérios de aceite

| Critério | Situação |
| --- | --- |
| Management Server acessível | Concluído |
| Host KVM registrado | Concluído |
| Storages disponíveis | Concluído, sujeito à confirmação no painel após reinício |
| System VMs `Running` | Em andamento |
| Instância de usuário em execução | Pendente |
| Rede isolada e NAT validados | Pendente |
| Volume e snapshot validados | Pendente |
| Multitenancy demonstrada | Pendente |
| Kubernetes/CSI demonstrado | Pendente |
| Resiliência documentada | Pendente |

## Evidências futuras

| Marco | Arquivo sugerido |
| --- | --- |
| System VMs em execução | `evidencias/04-cloudstack/E19-system-vms-running.png` |
| Capacidade KVM atualizada | `evidencias/03-kvm/E20-capacidade-kvm-atualizada.png` |
| Instância de teste | `evidencias/05-testes/E21-instancia-teste-running.png` |
| NAT/IP público | `evidencias/05-testes/E22-nat-ip-publico-validado.png` |
| Volume | `evidencias/05-testes/E23-volume-anexado.png` |
| Snapshot | `evidencias/05-testes/E24-snapshot-concluido.png` |
| Multitenancy | `evidencias/05-testes/E25-multitenancy-validado.png` |
| Kubernetes/CSI | `evidencias/05-testes/E26-kubernetes-csi-validado.png` |
| Resiliência | `evidencias/05-testes/E27-resiliencia-validada.png` |

## Links

- [Guia rápido de reprodução](01-guia-rapido-virtualbox.md)
- [Relatório técnico](../relatorio/RELATORIO-POC-CLOUDSTACK.md)
- [Evidências da POC](../evidencias/)
