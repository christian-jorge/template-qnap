# QNAP NAS SNMP Template for Zabbix 7.0

Template para monitoramento de QNAP NAS via SNMP, baseado na MIB QTS oficial da QNAP.

![Zabbix](https://img.shields.io/badge/Zabbix-7.0-red)
![SNMP](https://img.shields.io/badge/SNMP-v2c%20%7C%20v3-blue)
![License](https://img.shields.io/badge/License-MIT-green)

## 📋 Descrição

Este template foi desenvolvido para monitorar dispositivos QNAP NAS utilizando o protocolo SNMP. Ele utiliza o OID base `.1.3.6.1.4.1.55062` (enterprise ID oficial da QNAP) e é compatível com a MIB QTS versão 2.0.

## ✨ Funcionalidades

### Métricas de Sistema
- Modelo, Hostname, Serial Number, Firmware
- Uptime do sistema
- Uso de CPU (%)
- Temperatura da CPU e do Sistema
- Memória Total, Livre, Disponível, Usada e % de Uso
- Status da fonte de alimentação

### Métricas de Storage
- **Discos**: Status, Temperatura, Capacidade, Fabricante, Modelo, Tipo (HDD/SSD)
- **RAID**: Status, Nível, Capacidade
- **Storage Pools**: Capacidade, Espaço Livre, % de Uso, Status
- **Volumes**: Capacidade, Espaço Livre, % de Uso, Status
- **LUNs iSCSI**: Capacidade, % de Uso, Status

### Métricas de Hardware
- **Ventoinhas**: Velocidade (RPM)
- **UPS** (se conectado): Status, Carga da Bateria, % de Carga

## 🚨 Triggers de Alerta

| Componente | Condição | Severidade |
|------------|----------|------------|
| CPU | Uso > 90% (média 5min) | ⚠️ Warning |
| CPU Temperatura | > 70°C | ⚠️ Warning |
| CPU Temperatura | > 80°C | 🔴 High |
| Sistema Temperatura | > 50°C | ⚠️ Warning |
| Sistema Temperatura | > 60°C | 🔴 High |
| Memória | Uso > 90% (média 5min) | ⚠️ Warning |
| Fonte de Alimentação | Falha detectada | 🔥 Disaster |
| Disco | Status anormal | 🔴 High |
| RAID | Degradado ou com falha | 🔥 Disaster |
| Storage Pool | Uso > 80% | ⚠️ Warning |
| Storage Pool | Uso > 90% | 🔴 High |
| Storage Pool | Erro de status | 🔴 High |
| Volume | Uso > 80% | ⚠️ Warning |
| Volume | Uso > 90% | 🔴 High |
| Volume | Uso > 95% | 🔥 Disaster |
| Ventoinha | Parada (0 RPM) | 🔴 High |
| LUN iSCSI | Uso > 80% | ⚠️ Warning |
| LUN iSCSI | Uso > 90% | 🔴 High |

> **Nota**: Os triggers possuem dependências configuradas para evitar alertas duplicados.

## 📦 Requisitos

- Zabbix Server 7.0 ou superior
- QNAP NAS com firmware QTS
- SNMP habilitado no QNAP (v2c ou v3)

## 🔧 Configuração do QNAP

### Habilitar SNMP no QNAP

1. Acesse o **Painel de Controle** do QNAP
2. Navegue até **Rede e Serviços de Arquivos** → **SNMP**
3. Habilite o serviço SNMP
4. Configure a versão desejada:

#### SNMPv2c (mais simples)
- Defina uma community string (ex: `public` ou uma personalizada)

#### SNMPv3 (mais seguro)
- Crie um usuário (ex: `zabbix`)
- Configure o nível de segurança desejado:
  - **noAuthNoPriv**: Apenas nome de usuário
  - **authNoPriv**: Usuário + senha de autenticação
  - **authPriv**: Usuário + autenticação + criptografia

## 📥 Instalação do Template

### Método 1: Via Interface Web

1. No Zabbix, acesse **Data collection** → **Templates**
2. Clique em **Import** (canto superior direito)
3. Selecione o arquivo `qnap_template.yaml`
4. Clique em **Import**

### Método 2: Via API

```bash
curl -X POST "http://seu-zabbix/api_jsonrpc.php" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "configuration.import",
    "params": {
      "format": "yaml",
      "source": "'"$(cat qnap_template.yaml)"'"
    },
    "auth": "seu-token",
    "id": 1
  }'
```

## 🖥️ Configuração do Host

1. Acesse **Data collection** → **Hosts** → **Create host**
2. Preencha os campos:
   - **Host name**: Nome do seu QNAP
   - **Templates**: Selecione "QNAP NAS SNMP"
   - **Host groups**: Selecione ou crie um grupo

3. Na aba **Interfaces**, adicione uma interface SNMP:

### Para SNMPv2c
| Campo | Valor |
|-------|-------|
| IP address | IP do QNAP |
| Port | 161 |
| SNMP version | SNMPv2 |
| SNMP community | Sua community string |

### Para SNMPv3
| Campo | Valor |
|-------|-------|
| IP address | IP do QNAP |
| Port | 161 |
| SNMP version | SNMPv3 |
| Security name | Seu usuário SNMPv3 |
| Security level | Conforme configurado no QNAP |
| Authentication/Privacy | Se aplicável |

## 🔍 Discovery Rules

O template inclui regras de descoberta automática para:

| Regra | Intervalo | Descrição |
|-------|-----------|-----------|
| Disk Discovery | 1h | Descobre todos os discos físicos |
| RAID Discovery | 1h | Descobre grupos RAID |
| Storage Pool Discovery | 1h | Descobre storage pools |
| Volume Discovery | 1h | Descobre volumes |
| Fan Discovery | 1h | Descobre ventoinhas |
| LUN Discovery | 1h | Descobre LUNs iSCSI |

## 📊 OIDs Utilizados

### Base OID
```
.1.3.6.1.4.1.55062.1 (enterprises.qnap.qts)
```

### Principais OIDs

| Categoria | OID | Descrição |
|-----------|-----|-----------|
| System | .1.3.6.1.4.1.55062.1.12.3 | Modelo |
| System | .1.3.6.1.4.1.55062.1.12.4 | Hostname |
| System | .1.3.6.1.4.1.55062.1.12.12 | CPU Usage |
| System | .1.3.6.1.4.1.55062.1.12.10 | CPU Temperature |
| System | .1.3.6.1.4.1.55062.1.12.11 | System Temperature |
| Memory | .1.3.6.1.4.1.55062.1.12.13 | Total Memory |
| Memory | .1.3.6.1.4.1.55062.1.12.16 | Used Memory |
| Storage | .1.3.6.1.4.1.55062.1.10.2 | Disk Table |
| Storage | .1.3.6.1.4.1.55062.1.10.5 | RAID Table |
| Storage | .1.3.6.1.4.1.55062.1.10.7 | Storage Pool Table |
| Storage | .1.3.6.1.4.1.55062.1.10.9 | Volume Table |

## 🐛 Troubleshooting

### Verificar conectividade SNMP

```bash
# SNMPv2c
snmpwalk -v2c -c public IP_DO_QNAP .1.3.6.1.4.1.55062

# SNMPv3 (noAuthNoPriv)
snmpwalk -v3 -u usuario -l noAuthNoPriv IP_DO_QNAP .1.3.6.1.4.1.55062

# SNMPv3 (authNoPriv)
snmpwalk -v3 -u usuario -l authNoPriv -a SHA -A "senha" IP_DO_QNAP .1.3.6.1.4.1.55062
```

### Problemas comuns

| Problema | Solução |
|----------|---------|
| Timeout | Verificar firewall (porta 161/UDP) |
| No response | Confirmar se SNMP está habilitado no QNAP |
| Authentication error | Verificar credenciais SNMPv3 |
| No data | Verificar se o OID é suportado pelo modelo do QNAP |

## 📁 Estrutura do Repositório

```
.
├── README.md
├── qnap_template.yaml      # Template principal
└── mibs/
    └── NAS.mib             # MIB QTS da QNAP (opcional)
```

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para:

1. Fazer um Fork do repositório
2. Criar uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Criar um Pull Request

## 📝 Changelog

### v1.0.0 (2025-01-28)
- Release inicial
- Suporte a monitoramento de sistema (CPU, Memória, Temperatura)
- Discovery automático de Discos, RAIDs, Pools, Volumes, Fans e LUNs
- Triggers de alerta configurados
- Compatível com Zabbix 7.0

## 📄 Licença

Este projeto está licenciado sob a licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.

## 🙏 Agradecimentos

- QNAP pela documentação da MIB QTS
- Comunidade Zabbix

---

**Autor**: Christian Andrei  
**Criado com auxílio de**: Claude AI (Anthropic)
