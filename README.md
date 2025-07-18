# 🖧 Servidor DNS com BIND9 – Ubuntu Server 22.04

📅 **Data de criação:** 18/07/2025  
🔧 **Autor:** Guilherme  
🛡️ **Função:** Servidor DNS autoritativo interno usando BIND9.

---

## 📌 Descrição

Este projeto configura um servidor DNS autoritativo com BIND9 para um domínio fictício `exemplo.local`, incluindo resolução direta e reversa, totalmente funcional em uma rede local.

---

## ✅ Requisitos

- Ubuntu Server 22.04
- Acesso root ou `sudo`
- IP estático recomendado (ex: `192.168.0.10`)

---

## 📦 Instalação

```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc dnsutils -y
```

---

## ⚙️ Configuração

### 1. Editar opções globais

```bash
sudo nano /etc/bind/named.conf.options
```

```conf
options {
    directory "/var/cache/bind";
    recursion no;
    allow-query { any; };
    listen-on { any; };
    listen-on-v6 { any; };
};
```

### 2. Adicionar zonas DNS

```bash
sudo nano /etc/bind/named.conf.local
```

```conf
zone "exemplo.local" {
    type master;
    file "/etc/bind/db.exemplo.local";
};

zone "0.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192";
};
```

### 3. Criar arquivo de zona direta

```bash
sudo cp /etc/bind/db.local /etc/bind/db.exemplo.local
sudo nano /etc/bind/db.exemplo.local
```

```dns
$TTL    604800
@       IN      SOA     ns1.exemplo.local. admin.exemplo.local. (
                             2         ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.exemplo.local.
@       IN      A       192.168.0.10
ns1     IN      A       192.168.0.10
www     IN      A       192.168.0.10
```

### 4. Criar arquivo de zona reversa

```bash
sudo cp /etc/bind/db.127 /etc/bind/db.192
sudo nano /etc/bind/db.192
```

```dns
$TTL    604800
@       IN      SOA     ns1.exemplo.local. admin.exemplo.local. (
                             2         ; Serial
                        604800         ; Refresh
                         86400         ; Retry
                       2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.exemplo.local.
10      IN      PTR     ns1.exemplo.local.
10      IN      PTR     www.exemplo.local.
```

---

## ✅ Validação

```bash
sudo named-checkconf
sudo named-checkzone exemplo.local /etc/bind/db.exemplo.local
sudo named-checkzone 0.168.192.in-addr.arpa /etc/bind/db.192
```

---

## 🔄 Reinício do serviço

```bash
sudo systemctl restart bind9
sudo systemctl enable bind9
```

---

## 🧪 Testes

```bash
dig @192.168.0.10 www.exemplo.local
dig -x 192.168.0.10
```

---

## 📓 Notas adicionais

Se quiser apontar a máquina local para usar este DNS:

```bash
sudo nano /etc/netplan/01-netcfg.yaml
```

```yaml
nameservers:
  addresses: [192.168.0.10]
```

```bash
sudo netplan apply
```
