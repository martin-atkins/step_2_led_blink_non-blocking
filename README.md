# step_2_led_blink_non-blocking
Building from step 1 with non-blocking

## Using these versions:
* `STM32CubeIDE` : `1.18.1`
* `STM32CubeMX` : `6.14.1`

## The right ways to blink without `HAL_Delay()`
* Option 1 — Timer interrupt
  * hardware timer generates periodic interrupts
  * LED toggles in ISR
  * CPU stays free
  * deterministic timing
* Option 2 — SysTick / timebase + state machine
  * poll `HAL_GetTick()`
  * non-blocking, but...
    * still tied to SysTick
 
**I’ll do Option 1: TIM2 interrupt**

### Part A — Configure TIM2 in CubeMX
* Open my `.ioc` file
  * **Enable TIM2**
    * Pinout & Configuration
    * Timers → TIM2
    * Set only
      * **Clock Source**: `Internal Clock`
    * Leave:
      * Slave Mode: `Disabled`
      * Trigger Source: `Disabled`
      * Channels 1–4: `Disabled`
      * Combined Channels: `Disabled`
      * One Pulse Mode: `unchecked`
  * **Configure TIM2 parameters**
    * Click **TIM2 → Configuration → Parameter Settings**
    * Set:
      * **Prescaler (PSC)**: `16000 - 1`
      * **Counter Period (ARR)**: `1000 - 1`
      * **Auto-reload preload**: `Enabled`
      * **Counter mode**: `Up`
    * Why?
      * `APB1` timer clock ≈ 16 MHz
      * Prescaler → 1 kHz
      * Period → 1 second
  * **Enable TIM2 interrupt**
    * Still in **TIM2 → Configuration:**
      * Go to **NVIC Settings**
      * Enable **TIM2 global interrupt**
  * **Save** to generate code
    * I should now have...
      * `MX_TIM2_Init()` generated
      * `htim2` declared
      * NVIC configured
### Coding updates
* In `main.c`:
  * **Start timer with interrupts**
    * inside `main()`:
      ```
      MX_TIM2_Init();
      HAL_TIM_Base_Start_IT(&htim2);
      ```
  * **Callback implementation**
    * add **outside** `main()`:
      ```
      void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
      {
          if (htim->Instance == TIM2)
          {
              HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
          }
      }
      ```
