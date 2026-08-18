# STM32 Low-Power RTC Wake-up Sensing
Energy-aware STM32 sensing firmware using RTC-triggered 60-second Stop 2 cycles, followed by 3-channel ADC/DMA burst acquisition with CPU sleep during data transfer.
This project demonstrates an energy-aware periodic sensing architecture implemented on an STM32L4 MCU (NUCLEO-L476RG Board).

The system uses the RTC wake-up timer to periodically wake the MCU from Stop 2 mode after approximately 60 seconds. Following wake-up, the system
restores the system clock and re-initializes the ADC before performing a 3-channel ADC acquisition using DMA.

During the ADC/DMA acquisition, the CPU enters Sleep mode using WFI, allowing DMA to continue transferring data while minimizing unnecessary
CPU activity. Once the DMA transfer is completed, the MCU wakes through the DMA interrupt, reorganizes the interleaved ADC buffer into individual
channel arrays, and proceeds to data processing or UART transmission.

## Experimental Power Measurement
The power-saving strategy was experimentally validated using current measurements of the STM32 sensing node.
The measurement captures two operating states within a complete sensing cycle:

| Operating State | MCU Current | Function |
|  STOP 2         | ~0 mA       | Periodic waiting with RTC wake-up |
| Sleep + ADC/DMA | ~7 mA       | Burst sensor acquisition with CPU sleeping |

During STOP 2, the MCU remains in a deep low-power state while the RTC provides the wake-up source. During ADC/DMA acquisition, the CPU enters
Sleep mode using WFI while DMA continues transferring ADC samples, reducing unnecessary CPU activity.
<img width="490" height="394" alt="图片" src="https://github.com/user-attachments/assets/a8d6bcc3-d374-4ad4-8e6b-fce1e21daa3e" />

Figure, Power Consumption During Low-Power Sensing Cycle

                    PERIODIC LOW-POWER SENSING
                              │
                              ▼
                 ┌─────────────────────────┐
                 │     RTC Wake-up Timer   │
                 │        60 seconds       │
                 └────────────┬────────────┘
                              │
                              ▼
              ╔══════════════════════════════╗
              ║          STM32 STOP 2        ║
              ║                              ║
              ║     Periodic low-power       ║
              ║         waiting state        ║
              ╚══════════════╤═══════════════╝
                             │
                       RTC Wake-up IRQ
                             │
                             ▼
                 ┌─────────────────────────┐
                 │ Restore System Clock    │
                 │ Re-initialize ADC      │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │       ADC + DMA         │
                 │                         │
                 │  3-channel acquisition  │
                 │            samples      │
                 └────────────┬────────────┘
                              │
                    DMA acquisition
                       in progress
                              │
                              ▼
                 ╔═════════════════════════╗
                 ║       CPU SLEEP         ║
                 ║         WFI             ║
                 ║                         ║
                 ║ DMA continues running   ║
                 ╚════════════╤════════════╝
                              │
                       DMA Complete IRQ
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  ADC Buffer Processing  │
                 │                         │
                 │  CH1 ← DMA[0,3,6...]   │
                 │  CH2 ← DMA[1,4,7...]   │
                 │  CH3 ← DMA[2,5,8...]   │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │ UART / Data Processing  │
                 └────────────┬────────────┘
                              │
                              ▼
                         NEXT CYCLE
                              │
                              └────→ RTC 60 s

