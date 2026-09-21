<div align="center">

<img src="assets/iris_banner.png" alt="IRIS Banner" width="100%"/>

# IRIS
### Intelligent Road Intersection System

**Sistema de semáforo inteligente con visión por computadora y aprendizaje por refuerzo**

[![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)](https://python.org)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.11-orange?logo=pytorch)](https://pytorch.org)
[![YOLO11](https://img.shields.io/badge/YOLO-11m-darkgreen?logo=ultralytics)](https://ultralytics.com)
[![PPO](https://img.shields.io/badge/RL-PPO-purple)](https://stable-baselines3.readthedocs.io)
[![CARLA](https://img.shields.io/badge/Simulator-CARLA_0.9.16-red)](https://carla.org)
[![Samsung](https://img.shields.io/badge/Samsung-Innovation_Campus-blue?logo=samsung)](https://samsung.com)

</div>

---

## ¿Qué es IRIS?

IRIS es un sistema de control de semáforos inteligente que combina **visión por computadora** y **aprendizaje por refuerzo** para reducir los tiempos de espera vehicular en intersecciones urbanas.

A diferencia de los semáforos tradicionales que operan con ciclos de tiempo fijo predefinidos, IRIS **observa el tráfico en tiempo real** a través de cámaras y **toma decisiones adaptativas** según las condiciones actuales de cada intersección.

> Desarrollado para el **Samsung Innovation Campus** como proyecto de inteligencia artificial aplicada a movilidad urbana.

---

## El problema

<div align="center">
<img src="assets/problema_trafico.png" width="700"/>
</div>

Los semáforos de **ciclo fijo** son el estándar en la mayoría de ciudades, pero presentan un problema fundamental: operan con tiempos predefinidos independientemente de si hay vehículos esperando o no. Esto genera:

- ⏱️ Tiempos de espera innecesarios en horas de bajo tráfico
- 🚗 Congestión acumulada en horas pico sin mecanismo de adaptación
- 🔄 Falta de coordinación entre intersecciones cercanas
- 🌍 Mayor emisión de CO₂ por vehículos detenidos sin necesidad

---

## La solución: IRIS

IRIS implementa un pipeline de tres etapas que opera en tiempo real:

```
Cámaras en poste
      │
      ▼
 ┌─────────────────┐
 │   iris.pt       │  ← YOLO11m fine-tuneado
 │  (Detección)    │     Detecta peatones, coches, motos y camiones
 └────────┬────────┘
          │ Conteos por carril
          ▼
 ┌─────────────────┐
 │  Agente PPO     │  ← Red neuronal entrenada con RL
 │  (Decisión)     │     2,000,000 pasos de entrenamiento
 └────────┬────────┘
          │ Fase óptima del semáforo
          ▼
 ┌─────────────────┐
 │   Semáforo      │  ← Control en tiempo real
 │   (Acción)      │     3 intersecciones simultáneas
 └─────────────────┘
```

---

## Arquitectura del sistema

<div align="center">
<img src="assets/arquitectura_sistema.png" width="800"/>
</div>

IRIS está compuesto por 4 módulos integrados:

### 🎥 CARLA Simulator
Entorno de simulación 3D que reproduce fielmente una red vial urbana con:
- Red de **3 intersecciones interconectadas** (Cruz de 4 calles + 2 ramales)
- **2 cámaras en postes** por intersección principal cubriendo todo el campo de visión
- Generación de tráfico dinámico con vehículos y peatones
- API de control de semáforos en tiempo real

### 👁️ iris.pt — Modelo de Visión
Modelo **YOLO11m** entrenado en dos fases sobre 233,957 imágenes para detección de tráfico desde ángulo de cámara en poste:

| Clase | Descripción |
|-------|-------------|
| 🚶 Peatón | Personas cruzando o esperando |
| 🚗 Coche | Automóviles de pasajeros |
| 🏍️ Moto | Motocicletas y bicicletas |
| 🚛 Camión | Autobuses, camiones y vans |

**Entrenado con 5 datasets especializados:**
- TUMTraf-I R2 — cámaras reales en infraestructura de intersección (IEEE Best Paper Award ITSC 2023)
- Intersection Flow 5K — intersecciones reales
- UA-DETRAC — tráfico vehicular variado
- WiderPerson — peatones urbanos
- VisDrone2019-DET — ángulo elevado similar a cámara en poste

**Pipeline de datos:** 94,497 imágenes originales → 233,957 imágenes con data augmentation (Albumentations)

<div align="center">
<img src="assets/yolo_detecciones.png" width="700"/>
</div>

### 🧠 Agente PPO — Controlador Inteligente
Agente de **Proximal Policy Optimization** entrenado para minimizar el tiempo de espera vehicular en toda la red:

- **Estado:** Vector de 28 valores normalizados por intersección (conteos vehiculares, fase activa, presión de tráfico)
- **Acciones:** 36 combinaciones de fases simultáneas para las 3 intersecciones
- **Recompensa:** Función compuesta que penaliza espera, congestión entre intersecciones y monopolio de fase
- **Entrenamiento:** 2,000,000 pasos con 6 perfiles de tráfico distintos (bajo, medio, alto, asimétrico, nocturno, crítico)

<div align="center">
<img src="assets/rl_resultados.png" width="700"/>
</div>

### 📊 Dashboard en Tiempo Real
Interfaz de monitoreo que visualiza el sistema completo operando:

<div align="center">
<img src="assets/dashboard_completo.png" width="800"/>
</div>

---

## Resultados

### Modelo de visión — iris.pt

Entrenamiento en dos fases sobre 233,957 imágenes:

| Fase | mAP@50 | mAP@50-95 | Precision | Recall |
|------|--------|-----------|-----------|--------|
| Transfer Learning (10 epochs) | 0.713 | 0.507 | 0.788 | 0.637 |
| **Fine-tuning — iris.pt (75 epochs)** | **0.768** | **0.571** | **0.830** | **0.692** |

**AP@50 por clase:**

| Clase | Transfer Learning | Fine-tuning (iris.pt) | Mejora |
|-------|------------------|----------------------|--------|
| 🚗 Coche | 0.923 | **0.938** | +1.6% |
| 🚛 Camión | 0.838 | **0.856** | +2.1% |
| 🚶 Peatón | 0.661 | **0.718** | +8.6% |
| 🏍️ Moto | 0.428 | **0.557** | **+30.1%** |

> La moto — el objeto más difícil por su tamaño reducido desde ángulo elevado — fue la clase con mayor mejora relativa tras el fine-tuning.

### Agente de RL — Comparativa

| Algoritmo | Resultado |
|-----------|-----------|
| Tiempo Fijo (baseline) | — |
| DQN | Supera al tiempo fijo |
| **PPO (IRIS)** | **Supera a DQN y al tiempo fijo** |

> PPO supera a DQN en espera promedio, espera máxima y throughput. Ambos superan significativamente al baseline de tiempo fijo en los 6 perfiles de tráfico evaluados.

---

## Stack tecnológico

<div align="center">

| Componente | Tecnología |
|-----------|-----------|
| Detección | YOLO11m (Ultralytics 8.4.23) |
| Entrenamiento RL | Stable Baselines3 2.7.1 — PPO |
| Simulación de tráfico | CityFlow |
| Simulación 3D | CARLA 0.9.16 |
| Augmentation | Albumentations 1.3.1 |
| Dashboard | Streamlit |
| Deep Learning | PyTorch 2.11 + CUDA 12.8 |
| Hardware | NVIDIA RTX 5060 Ti 16GB |

</div>

---

## Flujo de desarrollo

```
Etapa 1 — Datos
    5 datasets públicos → pipeline de fusión → 233,957 imágenes aumentadas

Etapa 2 — Visión (iris.pt)
    YOLO11m preentrenado (COCO)
    → Transfer Learning (backbone congelado, 10 epochs) → mAP50: 0.713
    → Fine-tuning (backbone libre, 75 epochs)           → mAP50: 0.768

Etapa 3 — Aprendizaje por Refuerzo
    CityFlow (simulador de tráfico)
    → Entorno personalizado (3 intersecciones, 6 perfiles de tráfico)
    → Entrenamiento PPO y DQN (2M pasos cada uno)
    → Resultado: PPO > DQN > Tiempo Fijo

Etapa 4 — Integración
    CARLA (simulación 3D) + iris.pt + Agente PPO + Dashboard Streamlit
```

---

## Sobre el proyecto

IRIS fue desarrollado como proyecto final del **Samsung Innovation Campus**, un programa de formación en Inteligencia Artificial. El proyecto está enmarcado dentro de la marca de desarrollo **Velantex** y forma parte del portafolio de proyectos de IA aplicada a ciudades inteligentes.

---

<div align="center">

**IRIS** · Ninety9 · Samsung Innovation Campus 2025-2026

*Intelligent Road Intersection System*

</div>
