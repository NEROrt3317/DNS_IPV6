# DNS_IPV6
Implementar un servidor dns con ipv4 &amp; ipv6

# Modificaciones para IPv6
- Agregar direcciones de escucha para IPv6: Debes especificar que el servidor DNS escuche también en direcciones IPv6 (en el puerto 53). Para esto, puedes agregar la dirección ::1 (localhost en IPv6) y también la dirección IPv6 de tu red, si la tienes asignada. Si no tienes una dirección IPv6 específica, puedes usar :: para escuchar en todas las interfaces IPv6 disponibles.

- Registros AAAA en la zona DNS: Si estás configurando un dominio (como grupo1.local), deberías agregar registros AAAA en el archivo de zona correspondiente para las direcciones IPv6 de tus hosts.

- Zona de Reverse DNS para IPv6: Si también necesitas realizar consultas reverse (PTR) para direcciones IPv6, deberás agregar la zona reverse correspondiente (con el prefijo ip6.arpa).

1. Configuración del archivo `/etc/named.conf con soporte IPv6`.

```
options {
    # IPv4
    listen-on port 53 { 127.0.0.1; 192.168.200.1; };
    
    # IPv6: escucha en localhost IPv6 (::1) y en todas las interfaces IPv6 (::) 
    listen-on-v6 port 53 { ::1; fe80::1; 2001:db8::1; };  # Agregar tus direcciones IPv6 aquí, si tienes una asignada

    directory     "/var/named";
    dump-file     "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    allow-query     { localhost; 192.168.200.0/24; };
    allow-transfer { none; };
    recursion yes;
    dnssec-enable yes;
    dnssec-validation yes;

    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";

    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";
};

logging {
    channel default_debug {
        file "data/named.run";
        severity dynamic;
    };
};

# Zona de DNS raíz
zone "." IN {
    type hint;
    file "named.ca";
};

# Zona directa para grupo1.local
zone "grupo1.local" IN {
    type master;
    file "db.grupo1.local";
    allow-update { none; };
};

# Zona reverse para IPv4
zone "200.168.192.in-addr.arpa" IN {
    type master;
    file "reverse.grupo1.local";
    allow-update { none; };
};

# Zona reverse para IPv6
zone "8.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
    type master;
    file "reverse.grupo1.local";
    allow-update { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```
## Explicación de las modificaciones:
`listen-on-v6`:
La línea listen-on-v6 hace que el servidor DNS también escuche en direcciones IPv6. Se especifican las siguientes direcciones:

`::1`: Dirección de loopback (localhost) para IPv6.
`fe80::1`: Dirección link-local para una posible configuración de red local.
`2001:db8::1`: Un ejemplo de una dirección IPv6 global (en este caso, ficticia para la demostración). Asegúrate de usar la dirección IPv6 correcta de tu servidor.
Zona reverse IPv6 (ip6.arpa):
Se añadió una zona reverse para las direcciones IPv6. Las direcciones IPv6 reverse se configuran en el espacio de nombres ip6.arpa. El ejemplo mostrado aquí es para una subred de IPv6 ficticia (2001:db8::/32). Debes reemplazarlo por el rango adecuado de tu red.

Archivos de zona para IPv6:
Asegúrate de agregar los registros AAAA en el archivo de zona correspondiente (db.grupo1.local o el archivo que estés usando para tus configuraciones directas).
2.	Editar el archivo de la zona que se referenció en el archivo anterior `sudo nano /var/named/db.grupo1.local`:
```
$TTL 86400
@   IN  SOA     ns1.grupo1.local. root.grupo1.local. (
        2020080801  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          ns1.grupo1.local.
@       IN  A           192.168.200.1
@       IN  A           192.168.200.2
@       IN  AAAA        2001:db8::1          ; Dirección IPv6 para el dominio
@       IN  AAAA        2001:db8::2          ; Dirección IPv6 para otro dominio

ns1     IN  A           192.168.200.1
ns1     IN  AAAA        2001:db8::1          ; Dirección IPv6 para ns1

client  IN  A           192.168.200.2
client  IN  AAAA        2001:db8::2          ; Dirección IPv6 para client
```
## Explicación de los cambios:
`Registros AAAA`: He agregado dos registros AAAA para @ (el dominio principal), uno para cada una de las direcciones IPv6 (2001:db8::1 y 2001:db8::2). Estos son los equivalentes a `los registros A` que ya tenías configurados para IPv4.

`Registros AAAA para ns1 y client`: También agregué registros AAAA para ns1.grupo1.local y client.grupo1.local para que tengan tanto una dirección IPv4 como una dirección IPv6.

_NOTA_:
Asegúrate de reemplazar las direcciones IPv6 2001:db8::1 y 2001:db8::2 por las direcciones IPv6 reales de tus servidores, ya que 2001:db8:: es una dirección de ejemplo reservada para documentación.
`Los registros AAAA` sirven para proporcionar la resolución de nombre a direcciones IPv6, de la misma manera que los registros A resuelven nombres a direcciones IPv4.
3.	Editar el archivo de la zona reversa que se referenció en el archivo anterior `sudo nano /var/named/reverse.grupo1.local`:
```
$TTL 86400
@   IN  SOA     ns1.grupo1.local. root.grupo1.local. (
        2023101201  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          ns1.grupo1.local.

; Registros PTR para IPv6
1       IN  PTR         ns1.grupo1.local.
2       IN  PTR         client.grupo1.local.

; Registros de A para IPv6
ns1     IN  AAAA        2001:db8::1
client  IN  AAAA        2001:db8::2
```
## Explicación de los cambios:
Formato de la zona reverse de IPv6:

Los registros PTR ahora se usan para resolver direcciones IPv6 en lugar de A.
He incluido la dirección 2001:db8::1 para ns1 y 2001:db8::2 para client, que son las direcciones IPv6 asociadas a los registros PTR en la zona reverse.
Zonas PTR:

Los registros PTR como 1 IN PTR ns1.grupo1.local. están invirtiendo la dirección 2001:db8::1, pero en este caso, como trabajamos con direcciones IPv6, la zona reverse para 2001:db8::1 sería 1.0.0.0.0.0.0.0.0.0.0.0.0.8.b.d.0.0.1.0.0.2.ip6.arpa, y así es como se crea la relación inversa.
Configuración de las direcciones AAAA:

Los registros AAAA son para resolver las direcciones IPv6 de los hosts, y en este caso hemos agregado registros AAAA tanto para ns1 como para client, con sus respectivas direcciones IPv6.
