# ThirdReality Zigbee 3.0 USB Dongle

## Introduction
---
The ThirdReality Zigbee 3.0 USB Dongle is based on the Bouffalo Lab **BL706** chip.  
- For firmware versions **v1.0 and above**, the dongle uses the **BLZ protocol** (recommended).  
- For firmware versions **below v1.0**, the dongle uses the **Zigate protocol** (legacy).  

## Firmware Versions
---
### Coordinator Firmware
Starting from **v1.0 and above**, the dongle **migrates to the BLZ protocol**.  
- Recommended for integration with **Zigbee2MQTT** and **Home Assistant**.  
- Contained `partition_cfg_2M.toml`, `R3_706_dongle.bin`, and `bl_factory_params_IoTKitA_32M.dts`.  
- Latest release: `v1.00.07`.  

#### Legacy (Firmware < v1.0)
- Used the **Zigate protocol**.  
- Contained `partition_cfg_2M.toml`, `R3_706_dongle.bin`, and `bl_factory_params_IoTKitA_32M.dts`.  
- Latest legacy release: `v0.00.21`.  

### Router Firmware (Beta)
- Contains `partition_cfg_2M.toml`, `R3_706_dongle.bin`, and `bl_factory_params_IoTKitA_32M.dts`.  
- Latest: `v0.00.16 (Beta)`.  

## Notice
---
1. Refer to the documentation in the `Doc-Flash-Guide` directory for flashing instructions.  
   - On **Ubuntu** or **Mac OS**, use flashing tool **1.9.0**, ensure the correct port is selected, and press the pin hole while powering on.  

## Protocol
---
- **BLZ protocol** (firmware ≥ v1.0): recommended and actively supported.  
BLZ protocol documentation: [link here](<https://github.com/bouffalolab/zigpy-blz/blob/main/docs/UG100%20Bouffalo%20Lab%20Zigbee%20(BLZ)%20Protocol.pdf>).  
- **Zigate protocol** (firmware < v1.0): legacy, no longer recommended.  

### Host Examples

#### zigpy-blz
- [zigpy-blz adapter](https://github.com/bouffalolab/zigpy-blz/tree/main#)  
- A **zigpy-based adapter** that allows BLZ protocol dongles to work with Home Assistant (ZHA).  

#### Custom zigbee-herdsman with BLZ
- [fangzheli/zigbee-herdsman](https://github.com/fangzheli/zigbee-herdsman)  
- A **zigbee-herdsman fork** with BLZ protocol support, enabling integration with **Zigbee2MQTT**.  

## Contribution
---
Right now, since **BLZ protocol support has not yet been merged** into the main branches of **ZHA** (Home Assistant) and **Zigbee2MQTT**, using it requires extra steps.  

The maintainers of ZHA and Zigbee2MQTT have indicated that **greater popularity and adoption of BLZ** will help drive inclusion into the upstream projects.  

👉 We encourage the community to:
- Try using the dongle with BLZ.  
- Share your feedback and experiences.  
- Join the discussion with project maintainers such as **@puddly**, **@Adminiuga** (Home Assistant), and **@Koenkk** (Zigbee2MQTT), so that BLZ protocol support can move closer to mainline adoption.  

Your contributions and visibility will help bring BLZ into the official ecosystem faster!  

## How To
---

There are **5 ways** to use the BLZ dongle, depending on your setup:

### Home Assistant (ZHA)

#### 1. HAOS / HA Supervised — via HACS (Recommended)

The easiest way. Install the custom ZHA component that adds BLZ radio support.

1. In Home Assistant, go to **HACS** > **Integrations**
2. Click three-dot menu > **Custom repositories**
3. Add `https://github.com/bouffalolab/haos_custom_zha_blz` with category **Integration**
4. Search for **ZHA with BLZ Radio Support** and click **Install**
5. Restart Home Assistant
6. Go to **Settings** > **Devices & Services** > **Add Integration** > **ZHA**
7. Select the BLZ dongle and choose **BLZ** as the radio type

- Repository: [bouffalolab/haos_custom_zha_blz](https://github.com/bouffalolab/haos_custom_zha_blz)

#### 2. HAOS / HA Supervised — Manual SSH Install

```bash
ssh -p 22 <user>@<HA_IP>
mkdir -p /config/custom_components && cd /config
git clone --depth 1 https://github.com/bouffalolab/haos_custom_zha_blz.git _tmp
cp -r _tmp/custom_components/zha custom_components/
rm -rf _tmp
ha core restart
```

#### 3. HA Core (Python venv)

Install zigpy-blz and the BLZ-patched ZHA library in your Home Assistant Core virtual environment:

```bash
source /path/to/ha-venv/bin/activate
pip install "zha @ git+https://github.com/bouffalolab/zha.git@feat/blz"
pip install "zigpy-blz @ git+https://github.com/bouffalolab/zigpy-blz.git@main"
```

Then copy the custom ZHA component into your HA config:
```bash
cd /path/to/ha-config
git clone --depth 1 https://github.com/bouffalolab/haos_custom_zha_blz.git _tmp
cp -r _tmp/custom_components/zha custom_components/
rm -rf _tmp
```

- Guide: [Home_Assistant_Core_BLZ_Radios_Integration_Guide](Howto/Home_Assistant_Core_BLZ_Radios_Integration_Guide.pdf)

### Zigbee2MQTT

#### 4. HAOS Add-on (Z2M in Home Assistant)

Install Zigbee2MQTT as a Home Assistant add-on with BLZ support:

1. Go to **Settings** > **Add-ons** > **Add-on Store** (three-dot menu)
2. Click **Repositories** and add:
   ```
   https://github.com/fangzheli/hassio-zigbee2mqtt
   ```
3. Refresh, find **Zigbee2MQTT** and click **Install**
4. In the add-on Configuration, set:
   - Serial port: `/dev/ttyUSB0` (or your dongle path)
   - Adapter: `blz`
   - Baud rate: `2000000`
5. Start the add-on

- Repository: [fangzheli/hassio-zigbee2mqtt](https://github.com/fangzheli/hassio-zigbee2mqtt)

#### 5a. Docker (Standalone Z2M)

```bash
docker run -d --name zigbee2mqtt-blz \
  --restart=unless-stopped \
  --device=/dev/ttyUSB0:/dev/ttyUSB0 \
  -p 8099:8099 \
  -v $(pwd)/z2m-data:/app/data \
  -e TZ=UTC \
  ghcr.io/fangzheli/zigbee2mqtt:latest-dev
```

Create `z2m-data/configuration.yaml` with adapter `blz` and baudrate `2000000`.

#### 5b. From Source (Standalone Z2M)

```bash
git clone -b feat/blz https://github.com/fangzheli/zigbee-herdsman.git
cd zigbee-herdsman && pnpm install && cd ..

git clone -b feat/blz-local-dev https://github.com/fangzheli/zigbee2mqtt.git
cd zigbee2mqtt && pnpm install && pnpm start
```

Configure `data/configuration.yaml` with adapter `blz` and baudrate `2000000`.

- Guide: [Building and Configuring Z2M with BLZ](Howto/Building%20and%20Configuring%20Zigbee2mqtt%20with%20BLZ%20Protocol%20Support.pdf)

## Buy
---
Amazon link: https://a.co/d/a319nqE

## FCC Statement
This device complies with part 15 of the FCC rules. Operation is subject to the following two conditions: (1) this device may not cause harmful interference, and (2) this device must accept any interference   received,   including   interference   that   may   cause undesired operation. 

Changes or modifications not expressly approved by the party responsible for compliance could 
void the user’s authority to operate the equipment. 

NOTE: This equipment has been tested and found to comply with the limits for a Class B digital device, pursuant to part 15 of the FCC Rules. These limits are designed to provide reasonable protection against harmful interference in a residential installation. This equipment generates uses and can radiate radio frequency energy and, if not installed and used in accordance with the instructions, may cause harmful interference to radio communications. However, there is no guarantee that interference will not occur in a particular installation. If this equipment does cause harmful interference to radio or television reception, which can be determined by turning the equipment off and on, the user is encouraged to try to correct the interference by one or more of the following measures:  

• Reorient or relocate the receiving antenna.  
• Increase the separation between the equipment and receiver.  
• Connect the equipment into an outlet on a circuit different from that to which the receiver is connected.  
‐ Consult the dealer or an experienced radio/TV technician for help important announcement 

# Important Note:
## Radiation Exposure Statement
This equipment complies with FCC radiation exposure limits set forth for an uncontrolled environment. This transmitter must not be co-located or operating in conjunction with any other antenna or transmitter.

## ISED Statement
‐ English: This device complies with Industry Canada license‐exempt RSS standard(s).Operation is subject to the following two conditions: (1) This device may not cause interference, and (2) This device must accept any interference, including interference that may cause undesired operation of the device.The digital apparatus complies with Canadian CAN ICES‐3 (B)/NMB‐3(B).

‐ French:Le présentappareilestconforme aux CNR d'Industrie Canada applicables aux appareils
radio exempts de licence. L'exploitationestautorisée aux deux conditions suivantes: (1) l'appareil ne doit pas produire de brouillage, et (2) l'utilisateur de l'appareildoit accepter tout brouillageradioélectriquesubi, mêmesi le brouillageest susceptible d'encompromettre le fonctionnement.

## Radiation Exposure Statement
This equipment complies with Canada radiation exposure limits set forth for an uncontrolled environment. This equipment should be installed and operated with minimum distance 20cm between the radiator & your body. 

## Déclarationd'exposition aux radiations
Cetéquipementestconforme Canada limitesd'exposition aux radiations dans un environnement non contrôlé. Cetéquipementdoitêtreinstallé et utilisé à distance minimum de 20cm entre le radiateur et votre corps. 