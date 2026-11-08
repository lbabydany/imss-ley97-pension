# Valuación Actuarial de Beneficios a Empleados (NIF D-3) — Caso SEAT

Este repositorio contiene el motor de cálculo actuarial en **Python** y los modelos en **Excel** desarrollados por la consultoría **La Pensión Factory** para la valuación de pasivos laborales y el diseño del Plan de Pensiones de la cartera de empleados de **SEAT** (ejercicios 2023–2024), bajo los lineamientos de la **Norma de Información Financiera D-3 (NIF D-3)**.

---

## 1. Alcance y Beneficios Valuados

El proyecto evalúa cuatro obligaciones contingentes sobre un censo activo de **327 empleados** (personal Sindicalizado y de Confianza):

| Beneficio | Elegibilidad | Condición de Pago (Decrementos) | Salario Base de Cálculo | Fórmula del Beneficio ($B_{x+t}$) |
| :--- | :--- | :--- | :--- | :--- |
| **1. Prima de Antigüedad** (Art. 162 LFT) | Todo el personal | Muerte ($m$), Invalidez ($i$), Despido ($d$), Separación Voluntaria ($sv$ con $AS \ge 15$) y Jubilación ($R$ a los 65 años) | Sueldo Base topado a $2\text{ SM}$ | $12 \text{ días} \times AS_{x+t}$ |
| **2. Indemnización por Despido** | Todo el personal | Despido injustificado ($d$) antes de la edad de retiro | Sueldo Integrado | $90 \text{ días} + 20 \text{ días} \times AS_{x+t}$ |
| **3. Indemnización al Retiro** | Personal Sindicalizado | Jubilación normal a los 65 años ($R$) | Sueldo Integrado | $90 \text{ días} + 20 \text{ días} \times AS_{x+t}$ |
| **4. Plan de Pensiones** | Personal elegible (65 años) | Jubilación normal a los 65 años ($R$) | Último Sueldo Integrado | Renta vitalicia del $3.00\% \times AS_{x+t}$ por año de servicio |

---

## 2. Hipótesis Actuariales y Financieras

### Hipótesis Demográficas (Decrementos Múltiples)
* **Mortalidad Activos y Pensionados ($q_x^{(m)}$):** Tabla de experiencia demográfica **EMSSA 09** (diferenciada por sexo: `EMSSAH-RCS-09` y `EMSSAM-RCS-09`).
* **Invalidez ($q_x^{(i)}$):** Tabla de tasas de invalidez **EISS-97**.
* **Rotación de Personal ($q_x^{(r)}$):** Tabla de **Winklevoss** calibrada a la experiencia histórica de bajas de la empresa (factor de ajuste $\alpha = 0.60$, con proporción de despido $\theta_d = 53.33\%$).
* **Jubilación ($q_x^{(R)}$):** Retiro determinista a la edad normal de jubilación $r = 65$ años ($q_{65}^{(R)} = 1$).

### Hipótesis Financieras (al 31 de diciembre de 2024)
* **Tasa de Descuento ($i$):** $11.34\%$ anual (determinada mediante curva de bonos gubernamentales y calce por Duración de Macaulay de los flujos proyectados $D \approx 15.06$ años).
* **Tasa de Incremento Salarial ($s$):** $8.00\%$ anual.
* **Tasa de Incremento al Salario Mínimo ($s_{SM}$):** $11.03\%$ anual (Salario Mínimo base 2024: $\$248.93\text{ MXN}$).
* **Tasa de Inflación de Largo Plazo ($\pi$):** $3.75\%$ anual.

---

## 3. Metodología Matemática (Método de Crédito Unitario Proyectado)

### Transición de Tasas Independientes a Probabilidades Dependientes
Dado un sistema de cuatro decrementos en competencia $(m, i, r, R)$, las tasas absolutas $q_{x+t}^{(k)}$ se transforman en probabilidades dependientes ${q'}_{x+t}^{(k)}$ bajo el supuesto de distribución uniforme de decrementos (UDD):

$${q'}_{x+t}^{(k)} = q_{x+t}^{(k)} \left[ 1 - \frac{1}{2} \sum_{j \neq k} q_{x+t}^{(j)} + \frac{1}{3} \sum_{j < l \neq k} q_{x+t}^{(j)} q_{x+t}^{(l)} - \frac{1}{4} \prod_{j \neq k} q_{x+t}^{(j)} \right]$$

La probabilidad de permanencia activa hasta la edad $x+t$ es:

$${}_t p_x^{(\tau)} = \prod_{u=0}^{t-1} \left( 1 - \sum_{k} {q'}_{x+u}^{(k)} \right)$$

### Valuación del Plan de Pensiones y Conmutados
El valor presente actuarial de la renta vitalicia mensual anticipada a la edad de retiro $r=65$ se obtiene mediante valores conmutados ($D_x = l_x v^x$, $N_x = \sum_{u=x}^{\omega} D_u$) y la aproximación de Woolhouse de primer orden:

$$\ddot{a}_{65}^{(12)} = \frac{N_{65}}{D_{65}} - \frac{11}{24}$$

El Valor Presente de las Obligaciones Totales ($\text{VPOT}_x$), la Obligación por Beneficios Definidos ($\text{OBD}_x$) y el Costo Laboral del Servicio Actual ($\text{CLSA}_x$) para un trabajador de edad $x$ y antigüedad actual $AS_x$ son:

$$\text{VPOT}_x^{(k)} = \sum_{t=0}^{65-x} B_{x+t}^{(k)} \cdot v^t \cdot {}_t p_x^{(\tau)} \cdot {q'}_{x+t}^{(k)}$$

$$\text{OBD}_x^{(k)} = \sum_{t=0}^{65-x} \text{VPOT}_{x+t}^{(k)} \cdot \left( \frac{AS_x}{AS_{x+t}} \right), \qquad \text{CLSA}_x^{(k)} = \sum_{t=1}^{65-x} \text{VPOT}_{x+t}^{(k)} \cdot \left( \frac{1}{AS_{x+t}} \right)$$

---

## 4. Estructura del Repositorio

```text
seat-actuarial-valuation-nifd3/
│
├── data/
│   ├── censo_2023.csv                # Plantilla Dic 2023 (338 empleados)
│   ├── censo_2024.csv                # Plantilla Nov/Dic 2024 (327 empleados)
│   ├── bajas_2022_2024.csv           # Histórico de bajas para calibración de rotación
│   └── tablas_biometricas.csv        # EMSSA-09, EISS-97, Winklevoss y Curva de Bonos
│
├── excel/
│   ├── Valuacion_Actuarial_Pension_Factory.xlsx
│
├── src/
│   ├── conmutados.py                 # Cálculo de tablas de vida y anualidades vitalicias
│   ├── decrementos.py                # Matriz de decrementos múltiples UDD
│   ├── valuacion_nifd3.py            # Motor de cálculo VPOT, OBD, CLSA y PE0 por beneficio
│   └── contabilidad_ori.py           # Conciliación contable NIF D-3, Cuentas T y ORI
│
├── docs/
│   └── nota_tecnica_seat.tex         # Nota técnica actuarial y auditoría metodológica en LaTeX
│
└── README.md
