---
title: Alv-tools, mi primera herramienta 🛠️
description: Os enseñaré mi primera herramienta creada con Bash, como instalarla, como funciona etc.
date: 2024-11-14
categories: [Vagrant, Pentesting, CTF]
tags: [Vagrant, Pentesting, CTF, Scripting]
img_path: "assets/img/alvtools.jpeg"
image: "assets/img/alvtools.jpeg"
---

# Cómo usar Vagrant para configurar un entorno de Pentesting seguro y replicable

## Última actualización de la herramienta

18-11-2024 -> He arreglado la herramienta ya que en algunos parámetros si no estaba instalado nmap, no lo instalaba automáticamente.

## Introducción

- Buenas a todos! 👋 Hoy os presento mi nueva herramienta creada en Bash. Antes de nada quiero recalcar que entiendo y comprendo que es una herramienta básica y que existen millones igual que estas y mejores.

- He acudido a ChatGPT ya que considero que es una buena herramienta de ayuda y con la que he aprendido mucho.

- Me lo he tomado como un **proyecto de superación** para aprender como funcionan las herramientas más conocidas, aunque no estén en el mismo lenguaje o utilizen diferentes metodologías, la lógica sigue siendo la misma.

- Esta herramienta se irá actualizando y estará en mi GitHub para que todos podamos contribuir. [😺GitHub](https://github.com/Alv-fh)

- Dicho esto, empezemos con la instalación y uso...

---
- [Cómo usar Vagrant para configurar un entorno de Pentesting seguro y replicable](#cómo-usar-vagrant-para-configurar-un-entorno-de-pentesting-seguro-y-replicable)
  - [Última actualización de la herramienta](#última-actualización-de-la-herramienta)
  - [Introducción](#introducción)
  - [Features](#features)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Demo](#demo)
    - [Manual](#manual)
    - [Port Scan](#port-scan)
    - [Service Enum](#service-enum)
    - [ARP Scan](#arp-scan)
    - [OS Detect](#os-detect)
    - [Full Scan](#full-scan)
  
---

## Features

Alv-tools allows for several common pentesting tasks:

- Port scanning
- Subnet ping sweep
- Service enumeration on specific IPs
- Network scanning (ARP scan)
- Operating system and version detection
- Comprehensive scan that includes ports, service versions, operating system, and domain name

## Installation

[🔗 Link Github - Alv-tools](https://github.com/Alv-fh/Alv-tools)

```bash
git clone https://github.com/Alv-fh/Alv-tools.git
cd Alv-tools
sudo bash alv-tools.sh -h
```

## Usage

```bash
sudo bash alv-tools.sh [OPTION] [TARGET]
```

## Demo

### Manual

![help](https://github.com/user-attachments/assets/63c6d66b-8f94-45fc-9fcf-f69cc3046ae0)

### Port Scan

![ports](https://github.com/user-attachments/assets/e2d6f38e-6d6e-44b6-9e7c-7c63a61bd9bf)

### Service Enum

![services](https://github.com/user-attachments/assets/6ff91b9e-d976-4227-a505-da37606925bc)

### ARP Scan

![arp-scan](https://github.com/user-attachments/assets/02c6dfdf-9037-45af-acb2-85eb3c7dbaee)

### OS Detect

![os](https://github.com/user-attachments/assets/e85f41b5-a27e-4ed7-9172-a7a5ed3c8ba4)

### Full Scan

![fullscan](https://github.com/user-attachments/assets/97d8acf1-524c-43d3-8e52-7af393450edd)

Goal for EJPTv2 > $249

<a href='https://ko-fi.com/W7W313M7FS' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://storage.ko-fi.com/cdn/kofi1.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>
