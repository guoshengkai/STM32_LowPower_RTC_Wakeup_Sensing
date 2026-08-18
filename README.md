Energy-Aware Sensing Workflow
<img width="490" height="394" alt="图片" src="https://github.com/user-attachments/assets/a8d6bcc3-d374-4ad4-8e6b-fce1e21daa3e" />


             ┌──────────────────────┐
             │      RTC Wake-up     │
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │      MCU Wake-up     │
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │    ADC + DMA         │
             │   Data Acquisition   │
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │   Data Processing    │
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │ UART / BLE Transfer  │
             └──────────┬───────────┘
                        ↓
             ┌──────────────────────┐
             │   Low-power Mode     │
             │   Sleep / Stop       │
             └──────────┬───────────┘
                        │
                        └──────→ RTC Wake-up

