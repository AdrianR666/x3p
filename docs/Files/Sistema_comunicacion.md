## Descripción, diagrama de bloques y tablas referentes a puertos e interfaces de comunicación de elementos

### Descripción de los puertos e interfaces
La placa de expansión del ROSMASTER X3 PLUS incluye una variedad de puertos e interfaces que permiten la comunicación y conexión de diferentes componentes. A continuación se describen los principales:

1. **Interfaz de alimentación DC 12V (T-shaped)**:  
   - **Tipo**: Entrada de alimentación.  
   - **Descripción**: Conectada a la fuente de alimentación o batería de 12V. Es la entrada principal de energía para la placa.  

2. **Salida de alimentación DC 12V**:  
   - **Tipo**: Salida de alimentación.  
   - **Descripción**: Proporciona energía de 12V a dispositivos externos.  

3. **Interfaz Micro USB**:  
   - **Tipo**: Comunicación y programación.  
   - **Descripción**: Usada para comunicación con la Jetson Nano y para programar el microcontrolador de la placa.  

4. **Interfaz Type-C**:  
   - **Tipo**: Alimentación.  
   - **Descripción**: Proporciona 5V/5A para alimentar dispositivos externos. No se utiliza para comunicación.  

5. **Interfaz I2C**:  
   - **Tipo**: Comunicación serial.  
   - **Descripción**: Conecta dispositivos I2C, como pantallas OLED o sensores adicionales.  

6. **Interfaz CAN**:  
   - **Tipo**: Comunicación.  
   - **Descripción**: Conecta dispositivos CAN, como sensores o actuadores compatibles.  

7. **Interfaz SBUS**:  
   - **Tipo**: Comunicación.  
   - **Descripción**: Conecta receptores de control remoto para operar el robot.  

8. **Interfaz PWM para servos**:  
   - **Tipo**: Control de servos.  
   - **Descripción**: Permite conectar servos de 5V o 6.8V. El voltaje se selecciona mediante un jumper.  

9. **Interfaz para motores**:  
   - **Tipo**: Control de motores.  
   - **Descripción**: Conecta hasta cuatro motores. El método de conexión varía según el modelo del robot.  

10. **Interfaz SWD**:  
    - **Tipo**: Depuración y programación.  
    - **Descripción**: Usada para depurar o programar el microcontrolador de la placa mediante herramientas como ST-Link o J-Link.  

### Diagrama de bloques
A continuación se muestra un diagrama de bloques que ilustra cómo se conectan los diferentes componentes a través de los puertos e interfaces de la placa de expansión:

+-------------------+       +-------------------+       +-------------------+
| Microcontrolador  |<----->| Puertos e        |<----->| Dispositivos      |
| de la placa      |       | interfaces       |       | externos         |
|  STM32 MCU       |       | (USB, I2C, CAN,  |       | (motores,        |
|                 |       | PWM, SBUS, etc.) |       | servos, sensores)|
+-------------------+       +-------------------+       +-------------------+
        ^                          ^                          ^
        |                          |                          |
        v                          v                          v
+-------------------+       +-------------------+       +-------------------+
|   Jetson Nano    |       |  Batería y        |       |  Fuente de poder  |
| (comunicación y  |       |  alimentación     |       |                   |
|   control)       |       |                   |       |                   |
+-------------------+       +-------------------+       +-------------------+


### Tabla de puertos e interfaces
| **Puerto/Interfaz** | **Tipo**       | **Descripción**                                                                 | **Componente Conectado**       |
|----------------------|----------------|---------------------------------------------------------------------------------|--------------------------------|
| DC 12V (T-shaped)   | Alimentación   | Entrada principal de energía (12V)                                              | Batería o fuente de poder      |
| Micro USB           | Comunicación   | Conexión con la Jetson Nano y programación del microcontrolador                 | Jetson Nano                    |
| Type-C              | Alimentación   | Proporciona 5V/5A para dispositivos externos                                    | Dispositivos externos          |
| I2C                 | Comunicación   | Conexión de dispositivos I2C (OLED, sensores)                                   | Pantallas OLED, sensores       |
| CAN                 | Comunicación   | Conexión de dispositivos CAN                                                    | Sensores o actuadores CAN      |
| SBUS                | Comunicación   | Conexión de receptores de control remoto                                        | Control remoto                 |
| PWM                 | Control        | Conexión de servos de 5V o 6.8V                                                | Servos                         |
| Motor               | Control        | Conexión de hasta cuatro motores                                                | Motores                        |
| SWD                 | Depuración     | Depuración y programación del microcontrolador                                  | ST-Link, J-Link                |


## Descripción de software, estructura de paquetes, librerías y programas de operación

### Comunicación serial
La comunicación serial es un método clave para la interacción entre la Jetson Nano y la placa de expansión del ROSMASTER X3 PLUS. A continuación se describe su configuración y funcionamiento:

#### Propósito de la comunicación serial
El objetivo principal de la comunicación serial es permitir el intercambio de datos entre la Jetson Nano y el microcontrolador STM32 de la placa de expansión. Esto incluye:
- Recepción de datos desde la Jetson Nano.
- Retransmisión de los datos recibidos (eco).
- Uso de la función `printf` redefinida para enviar datos a través del puerto serial.

#### Configuración del puerto serial (USART1)
El puerto USART1 se configura con los siguientes parámetros:
- **Modo**: Comunicación asíncrona.
- **Baud rate**: 115200 baudios.
- **Ancho de datos**: 8 bits.
- **Bit de parada**: 1 bit.
- **Control de flujo**: Deshabilitado.

#### Hardware involucrado
- **Micro-USB**: Conecta la placa de expansión a la Jetson Nano para la comunicación serial.
- **USART1**: Puerto serial utilizado para la transmisión y recepción de datos.

#### Flujo de trabajo
1. **Inicialización**: El puerto USART1 se inicializa con los parámetros mencionados.
2. **Recepción de datos**: El microcontrolador recibe datos a través de USART1.
3. **Retransmisión**: Los datos recibidos se envían de vuelta a la Jetson Nano.
4. **Interrupciones**: Se utiliza una interrupción para manejar la recepción de datos y evitar la pérdida de paquetes.

#### Código principal
A continuación se describe el núcleo del código utilizado para la comunicación serial:

```c
// Inicialización del USART1
void USART1_Init(void) {
    HAL_UART_Receive_IT(&huart1, (uint8_t *)&RxTemp, 1);
}

// Envío de un byte a través del USART1
void USART1_Send_U8(uint8_t ch) {
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);
}

// Envío de un array de datos a través del USART1
void USART1_Send_ArrayU8(uint8_t *BufferPtr, uint16_t Length) {
    #if ENABLE_UART_DMA
    HAL_UART_Transmit_DMA(&huart1, BufferPtr, Length);
    #else
    while (Length--) {
        USART1_Send_U8(*BufferPtr);
        BufferPtr++;
    }
    #endif
}

// Interrupción de recepción de datos
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    USART1_Send_U8(RxTemp);  // Retransmisión de los datos recibidos
    HAL_UART_Receive_IT(&huart1, (uint8_t *)&RxTemp, 1);  // Continuar recibiendo datos
}

// Redefinición de la función printf para usar USART1
#ifdef __GNUC__
#define PUTCHAR_PROTOTYPE int __io_putchar(int ch)
#else
#define PUTCHAR_PROTOTYPE int fputc(int ch, FILE *f)
#endif
PUTCHAR_PROTOTYPE {
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xFFFF);
    return ch;
}
```

#### Conexión hardware
- Cable micro-USB: Conecta la placa de expansión a la Jetson Nano.
- Puerto USART1: Utilizado para la comunicación serial entre el microcontrolador y la Jetson Nano.

#### Efecto experimental
Al programar el microcontrolador y conectar la placa de expansión a la Jetson Nano:

- El LED parpadea cada 200 ms.

- Cada vez que se presiona un botón, el buzzer suena y se envía un mensaje a través del puerto serial.

- Si se envía un carácter (por ejemplo, 'a') desde la Jetson Nano, el microcontrolador lo retransmite.

## Comunicación CAN
La comunicación CAN (Controller Area Network) es un protocolo robusto utilizado para la interconexión de dispositivos en entornos industriales y robóticos. A continuación se describe su configuración y funcionamiento en el ROSMASTER X3 PLUS:

#### Propósito de la comunicación CAN
El objetivo principal de la comunicación CAN en este robot es:
- Permitir el intercambio de datos entre el microcontrolador STM32 y dispositivos CAN externos.
- Enviar y recibir datos en modo **loopback** para pruebas internas.
- Utilizar interrupciones para manejar la recepción de datos y evitar la pérdida de paquetes.

#### Configuración del bus CAN
El bus CAN se configura con los siguientes parámetros:
- **Baud rate**: 1000 kbps (1 Mbps).
- **Modo**: Loopback (para pruebas internas) o Normal (para conexión con dispositivos externos).
- **Pines**: PB8 (CAN_RX) y PB9 (CAN_TX).

#### Hardware involucrado
- **Transceptor CAN**: SN65HVD230DR, que convierte los niveles lógicos del microcontrolador a niveles CAN.
- **Resistencia de terminación**: 120Ω, necesaria para evitar reflexiones de señal en el bus CAN.

#### Flujo de trabajo
1. **Inicialización**: El periférico CAN se configura con el baud rate y modo adecuados.
2. **Filtros**: Se configuran filtros para seleccionar los mensajes CAN que se desean recibir.
3. **Interrupciones**: Se habilita la interrupción de recepción para manejar los datos entrantes.
4. **Envío de datos**: Se envían mensajes CAN cuando se presiona un botón.
5. **Recepción de datos**: Los mensajes recibidos se imprimen en el asistente de puerto serial.

#### Código principal
A continuación se describe el núcleo del código utilizado para la comunicación CAN:

```c
// Inicialización del CAN
void Can_Init(void) {
    // Configuración del filtro CAN
    sFilterConfig.FilterBank = 0;
    sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
    sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
    sFilterConfig.FilterIdHigh = 0x0000;
    sFilterConfig.FilterIdLow = 0x0000;
    sFilterConfig.FilterMaskIdHigh = 0x0000;
    sFilterConfig.FilterMaskIdLow = 0x0000;
    sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
    sFilterConfig.FilterActivation = CAN_FILTER_ENABLE;

    // Aplicar la configuración del filtro
    if (HAL_CAN_ConfigFilter(&hcan, &sFilterConfig) != HAL_OK) {
        Error_Handler();
    }

    // Iniciar el periférico CAN
    if (HAL_CAN_Start(&hcan) != HAL_OK) {
        Error_Handler();
    }

    // Habilitar la notificación de recepción CAN
    if (HAL_CAN_ActivateNotification(&hcan, CAN_IT_RX_FIFO0_MSG_PENDING) != HAL_OK) {
        Error_Handler();
    }
}

// Función para enviar datos a través del CAN
void Can_Test_Send(void) {
    uint8_t TxData[8];
    uint32_t TxMailbox = 0;
    TxHeader.StdId = 0x000F;
    TxHeader.ExtId = 0x00;
    TxHeader.RTR = CAN_RTR_DATA;
    TxHeader.IDE = CAN_ID_STD;
    TxHeader.DLC = 8;
    TxHeader.TransmitGlobalTime = DISABLE;

    // Preparar datos de prueba
    for (int i = 0; i < 8; i++) {
        TxData[i] = 1 << i;
    }

    // Enviar datos a través del CAN
    if (HAL_CAN_AddTxMessage(&hcan, &TxHeader, TxData, &TxMailbox) != HAL_OK) {
        Error_Handler();
    }
}

// Interrupción de recepción CAN
void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan) {
    uint8_t RxData[8];
    if (HAL_CAN_GetRxMessage(hcan, CAN_RX_FIFO0, &RxHeader, RxData) != HAL_OK) {
        Error_Handler();
    } else {
        // Imprimir datos recibidos en el puerto serial
        printf("CAN Receive:%02X %02X %02X %02X %02X %02X %02X %02X\n",
               RxData[0], RxData[1], RxData[2], RxData[3],
               RxData[4], RxData[5], RxData[6], RxData[7]);
    }
}
```
## Control de servos PWM mediante interrupciones de temporizador
El control de servos PWM en el ROSMASTER X3 PLUS se realiza mediante interrupciones generadas por el temporizador TIM7 del microcontrolador STM32. A continuación se describe su configuración y funcionamiento:

#### Propósito del control de servos PWM
El objetivo principal es:
- Generar señales PWM para controlar la posición de los servos.
- Utilizar interrupciones de temporizador para simular la señal PWM con alta precisión.

#### Configuración del temporizador TIM7
El temporizador TIM7 se configura para generar interrupciones periódicas que permiten simular la señal PWM. Los parámetros de configuración son:
- **Prescaler**: 71 (para ajustar la frecuencia del temporizador).
- **Periodo del contador**: 9 (para ajustar la frecuencia de la interrupción).
- **Modo de contador**: Up (contador ascendente).
- **Precarga automática**: Habilitada.

#### Hardware involucrado
- **Servos PWM**: Conectados a los pines PC0 (S1), PC1 (S2), PC2 (S3) y PC3 (S4) del STM32.
- **Jumper de voltaje**: Permite seleccionar entre 5V y 6.8V para alimentar los servos.

#### Flujo de trabajo
1. **Inicialización**: El temporizador TIM7 se configura para generar interrupciones periódicas.
2. **Generación de PWM**: En cada interrupción, se actualiza el estado de los pines de control de los servos para simular la señal PWM.
3. **Control de posición**: Se ajusta el ángulo de los servos modificando el ciclo de trabajo de la señal PWM.

#### Código principal
A continuación se describe el núcleo del código utilizado para el control de servos PWM:

```c
// Inicialización de los servos PWM
void PwmServo_Init(void) {
    for (int i = 0; i < MAX_PWM_SERVO; i++) {
        g_pwm_angle[i] = 90;  // Inicializar todos los servos en 90 grados
        g_angle_num[i] = PwmServo_Angle_To_Pulse(g_pwm_angle[i]);
    }
}

// Conversión de ángulo a pulsos PWM
static uint16_t PwmServo_Angle_To_Pulse(uint8_t angle) {
    uint16_t pulse = (angle * 11 + 500) / 10;  // Fórmula para convertir ángulo a pulsos
    return pulse;
}

// Establecer el ángulo de un servo específico
void PwmServo_Set_Angle(uint8_t index, uint8_t angle) {
    if (index >= MAX_PWM_SERVO || angle > 180) return;
    g_pwm_angle[index] = angle;
    g_angle_num[index] = PwmServo_Angle_To_Pulse(angle);
}

// Establecer el ángulo de todos los servos
void PwmServo_Set_Angle_All(uint8_t angle_s1, uint8_t angle_s2, uint8_t angle_s3, uint8_t angle_s4) {
    if (angle_s1 <= 180) {
        g_pwm_angle[0] = angle_s1;
        g_angle_num[0] = PwmServo_Angle_To_Pulse(angle_s1);
    }
    if (angle_s2 <= 180) {
        g_pwm_angle[1] = angle_s2;
        g_angle_num[1] = PwmServo_Angle_To_Pulse(angle_s2);
    }
    if (angle_s3 <= 180) {
        g_pwm_angle[2] = angle_s3;
        g_angle_num[2] = PwmServo_Angle_To_Pulse(angle_s3);
    }
    if (angle_s4 <= 180) {
        g_pwm_angle[3] = angle_s4;
        g_angle_num[3] = PwmServo_Angle_To_Pulse(angle_s4);
    }
}

// Manejo de la señal PWM en la interrupción del temporizador
void PwmServo_Handle(void) {
    g_pwm_pulse++;
    if (g_pwm_pulse <= g_angle_num[0]) SERVO_1_HIGH(); else SERVO_1_LOW();
    if (g_pwm_pulse <= g_angle_num[1]) SERVO_2_HIGH(); else SERVO_2_LOW();
    if (g_pwm_pulse <= g_angle_num[2]) SERVO_3_HIGH(); else SERVO_3_LOW();
    if (g_pwm_pulse <= g_angle_num[3]) SERVO_4_HIGH(); else SERVO_4_LOW();
    if (g_pwm_pulse >= 2000) g_pwm_pulse = 0;  // Reiniciar el contador de pulsos
}

// Interrupción del temporizador TIM7
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM7) {
        PwmServo_Handle();  // Llamar a la función de manejo de PWM
    }
}
```
## Control de servos seriales
El control de servos seriales en el ROSMASTER X3 PLUS se realiza a través del puerto USART3 del microcontrolador STM32. A continuación se describe su configuración y funcionamiento:

#### Propósito del control de servos seriales
El objetivo principal es:
- Controlar la posición de los servos seriales mediante comandos enviados a través del puerto USART3.
- Leer la posición actual de los servos para realizar ajustes precisos.

#### Configuración del puerto USART3
El puerto USART3 se configura con los siguientes parámetros:
- **Modo**: Comunicación asíncrona.
- **Baud rate**: 115200 baudios.
- **Ancho de datos**: 8 bits.
- **Bit de parada**: 1 bit.
- **Pines**: PC10 (USART3_TX) y PC11 (USART3_RX).

#### Hardware involucrado
- **Servos seriales**: Conectados al puerto USART3 de la placa de expansión.
- **Fuente de alimentación**: Se recomienda utilizar una fuente de 12V para evitar sobrecargas en la placa.

#### Flujo de trabajo
1. **Inicialización**: El puerto USART3 se configura para la comunicación con los servos.
2. **Control de servos**: Se envían comandos para mover los servos a posiciones específicas.
3. **Lectura de posición**: Se solicita y se lee la posición actual de los servos.
4. **Interrupciones**: Se utiliza una interrupción para manejar la recepción de datos desde los servos.

#### Código principal
A continuación se describe el núcleo del código utilizado para el control de servos seriales:

```c
// Inicialización del USART3
void USART3_Init(void) {
    HAL_UART_Receive_IT(&huart3, (uint8_t *)&RxTemp, 1);
}

// Envío de un byte a través del USART3
void USART3_Send_U8(uint8_t ch) {
    HAL_UART_Transmit(&huart3, (uint8_t *)&ch, 1, 0xFFFF);
}

// Envío de un array de datos a través del USART3
void USART3_Send_ArrayU8(uint8_t *BufferPtr, uint16_t Length) {
    while (Length--) {
        USART3_Send_U8(*BufferPtr);
        BufferPtr++;
    }
}

// Control de un servo serial
void DartServo_Ctrl(uint8_t id, uint16_t value, uint16_t time) {
    uint8_t head1 = 0xff;
    uint8_t head2 = 0xff;
    uint8_t s_id = id & 0xff;
    uint8_t len = 0x07;
    uint8_t cmd = 0x03;
    uint8_t addr = 0x2a;

    if (value > MAX_PULSE) value = MEDIAN_VALUE;
    else if (value < MIN_PULSE) value = MEDIAN_VALUE;

    uint8_t pos_H = (value >> 8) & 0xff;
    uint8_t pos_L = value & 0xff;

    uint8_t time_H = (time >> 8) & 0xff;
    uint8_t time_L = time & 0xff;

    uint8_t checknum = (~(s_id + len + cmd + addr + pos_H + pos_L + time_H + time_L)) & 0xff;
    uint8_t data[] = {head1, head2, s_id, len, cmd, addr, pos_H, pos_L, time_H, time_L, checknum};

    USART3_Send_ArrayU8(data, sizeof(data));
}

// Solicitud de la posición actual de un servo
void DartServo_Get_Angle(uint8_t id) {
    uint8_t head1 = 0xff;
    uint8_t head2 = 0xff;
    uint8_t s_id = id & 0xff;
    uint8_t len = 0x04;
    uint8_t cmd = 0x02;
    uint8_t param_H = 0x38;
    uint8_t param_L = 0x02;

    uint8_t checknum = (~(s_id + len + cmd + param_H + param_L)) & 0xff;
    uint8_t data[] = {head1, head2, s_id, len, cmd, param_H, param_L, checknum};
    USART3_Send_ArrayU8(data, sizeof(data));
}

// Interrupción de recepción de datos
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    if (huart == &huart3) {
        DartServo_Revice(RxTemp_3);
        HAL_UART_Receive_IT(&huart3, (uint8_t *)&RxTemp_3, 1);
    }
}
```

## Captura de datos del encoder
La captura de datos del encoder en el ROSMASTER X3 PLUS se realiza mediante los temporizadores (TIM2, TIM3, TIM4 y TIM5) del microcontrolador STM32. A continuación se describe su configuración y funcionamiento:

#### Propósito de la captura de datos del encoder
El objetivo principal es:
- Capturar los pulsos generados por los encoders de los motores para determinar su velocidad y posición.
- Utilizar el modo **Encoder Mode** de los temporizadores para contar los pulsos de manera precisa.

#### Configuración de los temporizadores
Los temporizadores se configuran en modo **Encoder Mode** para capturar los pulsos de los encoders. A continuación se detalla la configuración:

1. **Temporizadores utilizados**:
   - **TIM2**: Encoder del motor M1.
   - **TIM3**: Encoder del motor M4.
   - **TIM4**: Encoder del motor M2.
   - **TIM5**: Encoder del motor M3.

2. **Modo de funcionamiento**:
   - **Encoder Mode**: Modo de cuadratura (T1 y T2), que permite contar los pulsos en ambos sentidos (adelante y atrás).
   - **Frecuencia de muestreo**: Cuádruple, lo que aumenta la precisión de la captura.

3. **Parámetros de configuración**:
   - **Prescaler**: 0 (sin división de la frecuencia del reloj).
   - **Periodo del contador**: 65535 (valor máximo para un contador de 16 bits).
   - **Filtro de entrada**: 0 (sin filtrado).

#### Hardware involucrado
- **Encoders**: Conectados a los canales 1 y 2 de los temporizadores TIM2, TIM3, TIM4 y TIM5.
- **Motores**: Cada motor (M1, M2, M3, M4) está asociado a un encoder específico.

#### Flujo de trabajo
1. **Inicialización**: Los temporizadores se configuran en modo **Encoder Mode** y se inician.
2. **Captura de pulsos**: Los temporizadores cuentan los pulsos generados por los encoders.
3. **Lectura de datos**: Los valores de los contadores se leen periódicamente para determinar la posición y velocidad de los motores.
4. **Actualización**: Los valores de los encoders se actualizan cada 10 ms para mantener la precisión.

#### Código principal
A continuación se describe el núcleo del código utilizado para la captura de datos del encoder:

```c
// Inicialización de los temporizadores en modo Encoder
void Encoder_Init(void) {
    HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_1 | TIM_CHANNEL_2);  // Motor M1
    HAL_TIM_Encoder_Start(&htim3, TIM_CHANNEL_1 | TIM_CHANNEL_2);  // Motor M4
    HAL_TIM_Encoder_Start(&htim4, TIM_CHANNEL_1 | TIM_CHANNEL_2);  // Motor M2
    HAL_TIM_Encoder_Start(&htim5, TIM_CHANNEL_1 | TIM_CHANNEL_2);  // Motor M3
}

// Lectura del contador del encoder
int16_t Encoder_Read_CNT(uint8_t Motor_id) {
    int16_t Encoder_TIM = 0;
    switch (Motor_id) {
        case MOTOR_ID_M1: Encoder_TIM = (int16_t)TIM2->CNT; TIM2->CNT = 0; break;
        case MOTOR_ID_M2: Encoder_TIM = (int16_t)TIM4->CNT; TIM4->CNT = 0; break;
        case MOTOR_ID_M3: Encoder_TIM = (int16_t)TIM5->CNT; TIM5->CNT = 0; break;
        case MOTOR_ID_M4: Encoder_TIM = (int16_t)TIM3->CNT; TIM3->CNT = 0; break;
        default: break;
    }
    return Encoder_TIM;
}

// Actualización de los valores de los encoders
void Encoder_Update_Count(void) {
    q_Encoder_M1_Now -= Encoder_Read_CNT(MOTOR_ID_M1);  // Motor M1
    q_Encoder_M2_Now += Encoder_Read_CNT(MOTOR_ID_M2);  // Motor M2
    q_Encoder_M3_Now += Encoder_Read_CNT(MOTOR_ID_M3);  // Motor M3
    q_Encoder_M4_Now -= Encoder_Read_CNT(MOTOR_ID_M4);  // Motor M4
}

// Obtención del valor actual del encoder
int Encoder_Get_Count_Now(uint8_t Motor_id) {
    if (Motor_id == MOTOR_ID_M1) return q_Encoder_M1_Now;
    if (Motor_id == MOTOR_ID_M2) return q_Encoder_M2_Now;
    if (Motor_id == MOTOR_ID_M3) return q_Encoder_M3_Now;
    if (Motor_id == MOTOR_ID_M4) return q_Encoder_M4_Now;
    return 0;
}

// Obtención de los valores de los cuatro encoders
void Encoder_Get_ALL(int* Encoder_all) {
    Encoder_all[0] = q_Encoder_M1_Now;  // Motor M1
    Encoder_all[1] = q_Encoder_M2_Now;  // Motor M2
    Encoder_all[2] = q_Encoder_M3_Now;  // Motor M3
    Encoder_all[3] = q_Encoder_M4_Now;  // Motor M4
}
```

## Obtención de datos del sensor de actitud ICM20948
El sensor de actitud de nueve ejes **ICM20948** es un componente clave en el ROSMASTER X3 PLUS, ya que proporciona datos de aceleración, giroscopio y magnetómetro. A continuación se describe su configuración y funcionamiento:

#### Propósito del sensor ICM20948
El objetivo principal es:
- Leer los datos brutos del sensor ICM20948 (acelerómetro, giroscopio y magnetómetro).
- Convertir los datos brutos en unidades físicas (g para el acelerómetro, dps para el giroscopio y µT para el magnetómetro).
- Imprimir los datos a través del puerto serial para su visualización y análisis.

#### Configuración del sensor ICM20948
El sensor ICM20948 se comunica con el microcontrolador STM32 a través del protocolo **SPI**. A continuación se detalla la configuración:

1. **Pines SPI**:
   - **MOSI (SDI)**: PB15.
   - **MISO (SDO)**: PB14.
   - **SCLK**: PB13.
   - **NSS (Chip Select)**: PB12 (controlado por software).

2. **Parámetros de configuración**:
   - **Modo SPI**: Full-Duplex Master.
   - **Tamaño de datos**: 8 bits.
   - **Baud rate**: 18.0 Mbps.
   - **Polaridad del reloj (CPOL)**: Low.
   - **Fase del reloj (CPHA)**: 1 Edge.

#### Hardware involucrado
- **Sensor ICM20948**: Integrado en la placa de expansión.
- **Conexiones SPI**: Conectado a los pines PB12, PB13, PB14 y PB15 del STM32.

#### Flujo de trabajo
1. **Inicialización**: Se resetea y configura el sensor ICM20948, incluyendo la selección de la fuente de reloj, la configuración del filtro y la calibración del giroscopio y el acelerómetro.
2. **Lectura de datos**: Se leen los datos brutos del acelerómetro, giroscopio y magnetómetro.
3. **Conversión de unidades**: Los datos brutos se convierten a unidades físicas (g, dps, µT).
4. **Envío de datos**: Los datos se envían a través del puerto serial para su visualización.

#### Código principal
A continuación se describe el núcleo del código utilizado para la obtención de datos del sensor ICM20948:

```c
// Inicialización del ICM20948
void ICM20948_init() {
    while (!ICM20948_who_am_i());  // Esperar a que el sensor esté listo
    ICM20948_device_reset();       // Resetear el sensor
    ICM20948_wakeup();             // Despertar el sensor
    ICM20948_clock_source(1);      // Seleccionar la fuente de reloj
    ICM20948_gyro_low_pass_filter(0);  // Configurar el filtro del giroscopio
    ICM20948_accel_low_pass_filter(0); // Configurar el filtro del acelerómetro
    ICM20948_gyro_calibration();   // Calibrar el giroscopio
    ICM20948_accel_calibration();  // Calibrar el acelerómetro
    ICM20948_gyro_full_scale_select(_2000dps);  // Seleccionar escala completa del giroscopio
    ICM20948_accel_full_scale_select(_16g);     // Seleccionar escala completa del acelerómetro
}

// Lectura de datos brutos del acelerómetro
void ICM20948_accel_read(raw_data_t* data) {
    uint8_t* temp = read_multiple_reg(ub_0, B0_ACCEL_XOUT_H, 6);
    data->x = (int16_t)(temp[0] << 8 | temp[1]);
    data->y = (int16_t)(temp[2] << 8 | temp[3]);
    data->z = (int16_t)(temp[4] << 8 | temp[5]) + g_scale_accel;  // Ajustar por gravedad
}

// Conversión de datos brutos a unidades g
void ICM20948_accel_read_g(axises_t* data) {
    ICM20948_accel_read(&g_raw_accel);
    data->x = (float)(g_raw_accel.x / g_scale_accel);
    data->y = (float)(g_raw_accel.y / g_scale_accel);
    data->z = (float)(g_raw_accel.z / g_scale_accel);
}

// Lectura de datos brutos del giroscopio
void ICM20948_gyro_read(raw_data_t* data) {
    uint8_t* temp = read_multiple_reg(ub_0, B0_GYRO_XOUT_H, 6);
    data->x = (int16_t)(temp[0] << 8 | temp[1]);
    data->y = (int16_t)(temp[2] << 8 | temp[3]);
    data->z = (int16_t)(temp[4] << 8 | temp[5]);
}

// Conversión de datos brutos a unidades dps
void ICM20948_gyro_read_dps(axises_t* data) {
    ICM20948_gyro_read(&g_raw_gyro);
    data->x = (float)(g_raw_gyro.x / g_scale_gyro);
    data->y = (float)(g_raw_gyro.y / g_scale_gyro);
    data->z = (float)(g_raw_gyro.z / g_scale_gyro);
}

// Lectura de datos brutos del magnetómetro
bool AK09916_mag_read(raw_data_t* data) {
    uint8_t* temp;
    uint8_t drdy = read_single_mag_reg(MAG_ST1) & 0x01;
    if (!drdy) return false;  // Verificar si hay datos nuevos
    temp = read_multiple_mag_reg(MAG_HXL, 6);
    data->x = (int16_t)(temp[1] << 8 | temp[0]);
    data->y = (int16_t)(temp[3] << 8 | temp[2]);
    data->z = (int16_t)(temp[5] << 8 | temp[4]);
    return true;
}

// Conversión de datos brutos a unidades µT
bool AK09916_mag_read_uT(axises_t* data) {
    bool new_data = AK09916_mag_read(&g_raw_mag);
    if (!new_data) return false;
    data->x = (float)(g_raw_mag.x * 0.15);
    data->y = (float)(g_raw_mag.y * 0.15);
    data->z = (float)(g_raw_mag.z * 0.15);
    return true;
}
```
## Control de motores
El control de los motores en el ROSMASTER X3 PLUS se realiza mediante los temporizadores **TIM1** y **TIM8** del microcontrolador STM32, que generan señales PWM para controlar los drivers de motor **AM2857**. A continuación se describe su configuración y funcionamiento:

#### Propósito del control de motores
El objetivo principal es:
- Controlar la rotación de los motores (hacia adelante, hacia atrás y detención).
- Utilizar señales PWM para ajustar la velocidad de los motores.
- Implementar funciones de parada libre y frenado.

#### Configuración de los temporizadores
Los temporizadores **TIM1** y **TIM8** se configuran para generar señales PWM en los siguientes canales:

1. **TIM1**:
   - **CH1**: Controla el motor M3 (arriba derecha).
   - **CH2**: Controla el motor M4 (abajo derecha).
   - **CH3**: Controla el motor M4 (abajo derecha).
   - **CH4**: Controla el motor M3 (arriba derecha).

2. **TIM8**:
   - **CH1**: Controla el motor M1 (arriba izquierda).
   - **CH2**: Controla el motor M1 (arriba izquierda).
   - **CH3**: Controla el motor M2 (abajo izquierda).
   - **CH4**: Controla el motor M2 (abajo izquierda).

#### Parámetros de configuración
- **Prescaler**: 0 (sin división de la frecuencia del reloj).
- **Periodo del contador**: 3600 (ajusta la frecuencia de la señal PWM).
- **Modo de contador**: Up (contador ascendente).
- **Precarga automática**: Habilitada.

#### Hardware involucrado
- **Drivers de motor AM2857**: Controlan la dirección y velocidad de los motores.
- **Motores**: Conectados a los drivers AM2857.
- **Fuente de alimentación**: Se recomienda utilizar una fuente de 12V para evitar sobrecargas.

#### Flujo de trabajo
1. **Inicialización**: Los temporizadores TIM1 y TIM8 se configuran para generar señales PWM.
2. **Control de velocidad**: Se ajusta el ciclo de trabajo de las señales PWM para controlar la velocidad de los motores.
3. **Control de dirección**: Se cambia la polaridad de las señales PWM para controlar la dirección de rotación.
4. **Parada y frenado**: Se implementan funciones para detener los motores de manera libre o frenada.

#### Código principal
A continuación se describe el núcleo del código utilizado para el control de motores:

```c
// Inicialización de los temporizadores PWM
void Motor_Init(void) {
    HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);  // Motor M3
    HAL_TIMEx_PWMN_Start(&htim1, TIM_CHANNEL_2);  // Motor M4
    HAL_TIMEx_PWMN_Start(&htim1, TIM_CHANNEL_3);  // Motor M4
    HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_4);  // Motor M3

    HAL_TIM_PWM_Start(&htim8, TIM_CHANNEL_1);  // Motor M1
    HAL_TIM_PWM_Start(&htim8, TIM_CHANNEL_2);  // Motor M1
    HAL_TIM_PWM_Start(&htim8, TIM_CHANNEL_3);  // Motor M2
    HAL_TIM_PWM_Start(&htim8, TIM_CHANNEL_4);  // Motor M2
}

// Función para detener los motores
void Motor_Stop(uint8_t brake) {
    if (brake != 0) brake = 1;
    PWM_M1_A = brake * MOTOR_MAX_PULSE;  // Motor M1
    PWM_M1_B = brake * MOTOR_MAX_PULSE;
    PWM_M2_A = brake * MOTOR_MAX_PULSE;  // Motor M2
    PWM_M2_B = brake * MOTOR_MAX_PULSE;
    PWM_M3_A = brake * MOTOR_MAX_PULSE;  // Motor M3
    PWM_M3_B = brake * MOTOR_MAX_PULSE;
    PWM_M4_A = brake * MOTOR_MAX_PULSE;  // Motor M4
    PWM_M4_B = brake * MOTOR_MAX_PULSE;
}

// Función para ajustar la velocidad de un motor
void Motor_Set_Pwm(uint8_t id, int16_t speed) {
    int16_t pulse = Motor_Ignore_Dead_Zone(speed);  // Ignorar la zona muerta
    if (pulse >= MOTOR_MAX_PULSE) pulse = MOTOR_MAX_PULSE;  // Limitar la velocidad máxima
    if (pulse <= -MOTOR_MAX_PULSE) pulse = -MOTOR_MAX_PULSE;

    switch (id) {
        case MOTOR_ID_M1:  // Motor M1
            pulse = -pulse;
            if (pulse >= 0) {
                PWM_M1_A = pulse;
                PWM_M1_B = 0;
            } else {
                PWM_M1_A = 0;
                PWM_M1_B = -pulse;
            }
            break;
        case MOTOR_ID_M2:  // Motor M2
            if (pulse >= 0) {
                PWM_M2_A = pulse;
                PWM_M2_B = 0;
            } else {
                PWM_M2_A = 0;
                PWM_M2_B = -pulse;
            }
            break;
        case MOTOR_ID_M3:  // Motor M3
            if (pulse >= 0) {
                PWM_M3_A = pulse;
                PWM_M3_B = 0;
            } else {
                PWM_M3_A = 0;
                PWM_M3_B = -pulse;
            }
            break;
        case MOTOR_ID_M4:  // Motor M4
            if (pulse >= 0) {
                PWM_M4_A = pulse;
                PWM_M4_B = 0;
            } else {
                PWM_M4_A = 0;
                PWM_M4_B = -pulse;
            }
            break;
    }
}
```
## Control de la barra de luces RGB
La barra de luces RGB en el ROSMASTER X3 PLUS se controla mediante el protocolo de comunicación del módulo **WS2812B**, utilizando el puerto **SPI3** del microcontrolador STM32. A continuación se describe su configuración y funcionamiento:

#### Propósito del control de la barra de luces RGB
El objetivo principal es:
- Controlar el color y los efectos de la barra de luces RGB.
- Simular el protocolo de comunicación del WS2812B utilizando el puerto SPI.
- Mostrar efectos visuales como cambios de color y secuencias.

#### Configuración del puerto SPI3
El puerto SPI3 se configura para simular el protocolo de comunicación del WS2812B. A continuación se detalla la configuración:

1. **Pines SPI**:
   - **MOSI (PB5)**: Pin de salida de datos para controlar la barra de luces RGB.

2. **Parámetros de configuración**:
   - **Modo SPI**: Transmit Only Master (solo transmisión).
   - **Tamaño de datos**: 8 bits.
   - **Baud rate**: 2.25 Mbps.
   - **Polaridad del reloj (CPOL)**: Low.
   - **Fase del reloj (CPHA)**: 2 Edge.

3. **DMA**: Se habilita el canal DMA2 para la transmisión de datos, lo que permite una comunicación eficiente y sin bloqueos.

#### Hardware involucrado
- **Barra de luces RGB**: Conectada al pin PB5 del STM32.
- **Protocolo WS2812B**: Utilizado para controlar los LEDs RGB individualmente.

#### Flujo de trabajo
1. **Inicialización**: Se configura el puerto SPI3 y se inicializa la estructura de datos para almacenar los colores de los LEDs.
2. **Configuración de colores**: Se establecen los colores de los LEDs individualmente o de toda la barra.
3. **Actualización**: Se envían los datos a la barra de luces RGB utilizando DMA para refrescar los colores.
4. **Efectos visuales**: Se implementan efectos como cambios de color y secuencias.

#### Código principal
A continuación se describe el núcleo del código utilizado para el control de la barra de luces RGB:

```c
// Estructura para almacenar los datos de los LEDs
typedef struct {
    union {
        uint8_t Buff[9];
        struct {
            uint8_t G[3];  // Verde (primero)
            uint8_t R[3];  // Rojo (segundo)
            uint8_t B[3];  // Azul (tercero)
        } RGB;
    } String[MAX_RGB];  // MAX_RGB es el número máximo de LEDs
} ws2812_t;

// Inicialización de la barra de luces RGB
void RGB_Init(void) {
    // Configuración inicial del SPI y DMA
    HAL_SPI_Init(&hspi3);
}

// Establecer el color de un LED individual
void RGB_Set_Color(uint8_t index, uint8_t r, uint8_t g, uint8_t b) {
    uint32_t color = r << 16 | g << 8 | b;  // Combinar los valores de color
    RGB_Set_Color_U32(index, color);  // Llamar a la función de configuración de color
}

// Establecer el color de un LED utilizando un valor de 32 bits
void RGB_Set_Color_U32(uint8_t index, uint32_t color) {
    if (index < MAX_RGB) {
        WS2812_Set_Color_One(index, color);  // Configurar el color de un LED
    } else if (index == RGB_CTRL_ALL) {
        for (uint16_t i = 0; i < MAX_RGB; i++) {
            WS2812_Set_Color_One(i, color);  // Configurar el color de todos los LEDs
        }
    }
}

// Actualizar la barra de luces RGB
void RGB_Update(void) {
    WS2812_Send_Data((uint8_t*)&g_ws2812.String[0].Buff, 9 * MAX_RGB);  // Enviar datos a la barra de luces
}

// Enviar datos utilizando DMA
static void WS2812_Send_Data(uint8_t *buf, uint16_t buf_size) {
    HAL_SPI_Transmit_DMA(&hspi3, buf, buf_size);  // Transmitir datos mediante DMA
}
```
