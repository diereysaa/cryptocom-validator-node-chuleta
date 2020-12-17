# Índice

- [Delegar CROs a un nodo](https://github.com/diereysaa/cryptocom-validator-node-chuleta/new/main#delegar-cros-a-un-nodo)
- [Comprobar cantidad delegada](https://github.com/diereysaa/cryptocom-validator-node-chuleta/new/main#comprobar-la-cantidad-delegada)
- [Recuperar CROs de un nodo al que le delegaste](https://github.com/diereysaa/cryptocom-validator-node-chuleta/new/main#recuperar-cros-de-un-nodo-al-que-le-delegaste)
- [Auto-delegación automática](https://github.com/diereysaa/cryptocom-validator-node-chuleta/new/main#auto-delegaci%C3%B3n-autom%C3%A1tica)
- [Transferir CROs de una cuenta a otra](https://github.com/diereysaa/cryptocom-validator-node-chuleta/new/main#transferir-cros-de-una-cuenta-a-otra)
- [Comprobar el saldo de una cuenta](https://github.com/diereysaa/cryptocom-validator-node-chuleta/new/main#comprobar-el-saldo-de-una-cuenta)

---

## Notas previas

>### :information_source: Qué es la delegación, recompensas, etc?
>
>La delegación de tokens es un préstamo que le haces al nodo. Para entendernos, vamos a pensar en personas y nodos. 
>
>Cuando Leslie te manda los tCROs (o ya pensando en la mainnet, cuando tú tienes los CROs) los tCROs/CROs son tuyos, de tu persona.
>
>Para crear un nodo, tú le prestas (delegas) a tu nodo una cantidad de tCROs/CROs (normalmente hacíamos esto con todos los tCROs disponibles)
>
>El nodo entonces empieza a funcionar, a firmar bloques y a :heart_eyes: **generar recompensas** :dollar:
>
>Estas recompensas son automáticamente enviadas a tu cuenta personal desde el nodo. Ahí tu puedes elegir si quedártelas (y gastar tus tCROs/CROs en lo que sea) o re-invertirlas (re-delegarlas).
>
>**Nota: Piensa que esto también se puede hacer desde tu cuenta personal a CUALQUIER nodo...**

---

## Delegar CROs a un nodo

Con este comando, podrás delegar los tCROs/CROs que tengas como persona a un nodo que no sea tuyo. Esto es bueno por si no quieres tener que preocuparte de mantener el nodo online, no quieres liarte con mantenimientos de servidores y otras cosas por el estilo.

Esto es de sentido común, pero piensa que estás poniendo en juego tus tCROs/CROs en el nodo de otro, así que hazlo sólo con alguien en quien confíes. 

Ojo, tampoco es que ese alguien se vaya a escapar con tus tCROs/CROs, pero sí tienes que tener en cuenta que si el nodo es penalizado (por tiempo offline o algo así) tu también perderás parte de tus tCROs/CROs.

Para delegar tus tCROs/CROs a otro nodo, ejecuta esto:

```shell
./chain-maind tx staking delegate [VALIDATOR-ADDRESS] [AMOUNT] --from=[LOCAL-KEY] --chain-id="testnet-croeseid-1" --gas-prices 0.1basetcro
```
Parámetros:
- `[VALIDATOR-ADDRESS]` es la dirección pública del nodo validador. Puedes irte al explorer, buscar tu nodo preferido y sacar de ahí la "Operator Address" (que empieza por "tcrocncl")
- `[AMOUNT]` es la cantidad de tCROs/CROs que quieres delegar. Recuerda poner el nombre de la moneda, ej. "100tcro"
- `[LOCAL-KEY]` es el nombre de tu cuenta, que puedes sacarlo haciendo `./chain-maind keys list`
- `--chain-id="testnet-croeseid-1"` ahora mismo estamos en la testnet, así que asegúrate de que esto es correcto si hay otra red (mainnet)

Cuando lo ejecutes, te pedirá tu passphrase. Confirma la transacción pulsando **"Y"** y ya se habrán delegado tus tCROs/CROs 

> :raising_hand: - Recuerda! Las recompensas por estos tCROs/CROs que has delegado volverán a tu cuenta automáticamente. Quizás sea buena idea auto-delegarlos automáticamente.

---

## Comprobar la cantidad delegada

En un momento dado, puedes querer comprobar cuánto has delegado en un nodo, para saber cuánto tienes, básicamente. 

Para ello, ejecuta:
```shell
./chain-maind query staking delegation [DELEGATOR-ADDRESS] [VALIDATOR-ADDRESS]
```
Parámetros:
- `[DELEGATOR-ADDRESS]` es tu propia dirección, que conseguiste al configurar el `chain-maind`
- `[VALIDATOR-ADDRESS]` es la dirección (`tcrocncl....`) del nodo al que le delegaste tus tCROs/CROs

---

## Recuperar CROs de un nodo al que le delegaste

Llegado el caso, querrás recuperar los tCROs/CROs del nodo al que se los delegaste. 

Para ello, ejecuta:
```shell
./chain-maind tx staking unbond [VALIDATOR-ADDRESS] [AMOUNT] --from=[LOCAL-KEY] --chain-id="testnet-croeseid-1" --gas-prices 0.1basetcro
```
Parámetros:
- `[VALIDATOR-ADDRESS]` es la dirección (`tcrocncl....`) del nodo al que le delegaste tus tCROs/CROs
- `[AMOUNT]` es la cantidad de tCROs/CROs que quieres recuperar. Recuerda poner el nombre de la moneda, ej. "100tcro"
- `[LOCAL-KEY]` es el nombre de tu cuenta, que puedes sacarlo haciendo `./chain-maind keys list`
- `--chain-id="testnet-croeseid-1"` ahora mismo estamos en la testnet, así que asegúrate de que esto es correcto si hay otra red (mainnet)

---

## Auto-delegación automática

Mediante estos comandos, vas a poder auto-delegar tus recompensas directamente a un nodo

### Pasos para la auto-delegación

##### 1.- Tienes que crearte un archivo (`node-auto-delegate-rewards.sh`) con este contenido:
```shell
#! /bin/bash
while true; do export ACCOUNT=[ACCOUNT NAME]; export PASS="[PASSWORD]"; export CHAIN=testnet-croeseid-1; export NODE="http://127.0.0.1:26657"; VALOPER=$(echo $PASS | ${HOME}/chain-maind keys show $ACCOUNT --bech val --address); NODETODELEGATE=VALOPER; echo $PASS | ${HOME}/chain-maind tx distribution withdraw-rewards $VALOPER --from $ACCOUNT --commission --chain-id $CHAIN --gas-prices 0.1basetcro --node $NODE --yes; sleep 30; value=$(${HOME}/chain-maind query bank balances $(echo $PASS | ${HOME}/chain-maind keys show $ACCOUNT --address) --chain-id $CHAIN --node $NODE -o json | jq -r '.balances | .[].amount'); value=$(($value - 500000)); echo $PASS | ${HOME}/chain-maind tx staking delegate $NODETODELEGATE ${value}basetcro --from $ACCOUNT --chain-id $CHAIN --node $NODE --gas-prices 0.1basetcro --yes; sleep 300; done
```
En este archivo hay que tener en cuenta varias cosas:
- `do export ACCOUNT=[ACCOUNT NAME]` Aquí tienes que cambiar [ACCOUNT NAME] por el nombre de tu cuenta (es el que pusiste al inicializar el `chain-maind`, puedes sacarlo haciendo `./chain-maind keys list`)
- `export PASS="[PASSWORD]"` Aquí tienes que cambiar [PASSWORD] por la passphrase que usas cuando haces cualquier acción con el `chain-maind`
- `${HOME}/chain-maind` esto aparece en varias partes del archivo, y hay que ponerlo de acuerdo a dónde tengas el ejecutable `chain-maind`. Lo de `${HOME}` hace referencia a tu directorio "home" que suele ser `/home/ubuntu`
- `NODETODELEGATE=VALOPER` esto dice que la delegación sea a tu propio nodo, pero en algunas circunstancias, querrás delegar tus tCROs/CROs a otro nodo, si es así, pon aquí el `tcrocncl....` del nodo al que quieres delegar tus recompensas de tCROs/CROs
---

##### 2.- Ahora, tienes que marcar el archivo como ejecutable:
```shell
chmod +x node-auto-delegate-rewards.sh
```

---

##### 3.- A continuación, tienes que crear un archivo (/etc/systemd/system/node-auto-delegate-rewards.service) con este contenido:
```shell
[Unit]
Description=Crypto.com croeseid rewards manager
After=network-online.target

[Service]
Type=simple
Restart=always
RestartSec=1
User=ubuntu
LimitNOFILE=65536
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/node-auto-delegate-rewards.sh

[Install]
WantedBy=multi-user.target
```
En este archivo, de nuevo, hay que comprobar que los directorios sean correctos, dependiendo de dónde tienes el `chain-maind`

---

##### 4.- Ahora tienes que lanzar el servicio con: 
```shell
sudo systemctl start node-auto-delegate-rewards.service
```

---

##### 5.- Por último, si quieres comprobar como va el log, puedes hacerlo con: 
```shell
journalctl -u node-auto-delegate-rewards -f
```

---

## Transferir CROs de una cuenta a otra

Si necesitas transferir CROs de una cuenta a otra puedes ejecutar esto: 
```shell
chain-maind tx bank send [SENDING-ADDRESS] [RECEIVING-ADDRESS] [AMOUNT] --chain-id="testnet-croeseid-1" --gas-prices 0.1basetcro
```
Parámetros:
- `[SENDING-ADDRESS]` es la dirección (`tcro....`) desde la que quieres mandar tus tCROs/CROs (debe ser tuya, por supuesto)
- `[RECEIVING-ADDRESS]` es la dirección (`tcro....`) a la que quieres REGALAR/mandar tus tCROs/CROs
- `[AMOUNT]` es la cantidad de tCROs/CROs que quieres mandar. Recuerda poner el nombre de la moneda, ej. "100tcro"
- `--chain-id="testnet-croeseid-1"` ahora mismo estamos en la testnet, así que asegúrate de que esto es correcto si hay otra red (mainnet)

---

## Comprobar el saldo de una cuenta

Para saber el saldo que hay en una cuenta (tuya o de otros) simplemente ejecuta esto:
```shell
chain-maind query bank balances [ADDRESS] --output json | jq
```
Parámetros:
- `[ADDRESS]` es la dirección (`tcro....`) de la que quieres consultar la cantidad de tCROs/CROs

