# freeRtos-project

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
This code sets up an STM32 microcontroller with FreeRTOS to manage multiple tasks, including toggling LEDs and handling button presses. It also initializes various peripherals and configures the system clock
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

This tutorial will help you understand how the code works and how to use it in your own projects.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overview
This code is for an STM32 microcontroller project that uses FreeRTOS to manage multiple tasks. It includes initialization of various peripherals (I2C, I2S, SPI, GPIO), system clock configuration, and task creation for controlling LEDs based on button input.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
Step-by-Step Breakdown
1. Includes and Definitions
c
#include "main.h"
#include "cmsis_os.h"
#include "usb_host.h"
These lines include the necessary header files for the project, including the main header, CMSIS-RTOS (FreeRTOS), and USB host functionality.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
2. Private Variables
c
I2C_HandleTypeDef hi2c1;
I2S_HandleTypeDef hi2s3;
SPI_HandleTypeDef hspi1;
osThreadId defaultTaskHandle;
TaskHandle_t myTask1Handle = NULL;
TaskHandle_t myTask2Handle = NULL;
These variables are used to handle different peripherals (I2C, I2S, SPI) and tasks.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
3. Function Prototypes
c
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
static void MX_I2C1_Init(void);
static void MX_I2S3_Init(void);
static void MX_SPI1_Init(void);
void StartDefaultTask(void const * argument);
These are the prototypes for the functions used to configure the system clock, initialize peripherals, and start the default task.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
4. Task Functions
Led_GateKeeper
c
void Led_GateKeeper(void *pvParameters)
{
    TickType_t xDelay = 1000 / portTICK_PERIOD_MS;
    for(;;)
    {
        HAL_GPIO_WritePin(GPIOD, LD4_Pin|LD3_Pin ,GPIO_PIN_RESET);
        vTaskDelay(xDelay);
        HAL_GPIO_WritePin(GPIOD, LD4_Pin|LD3_Pin ,GPIO_PIN_SET);
        vTaskDelay(xDelay);
    }
}
This task toggles LEDs LD4 and LD3 on and off every second.

Led_GateKeeper1
c
void Led_GateKeeper1(void *pvParameters)
{
    TickType_t xDelay = 1000 / portTICK_PERIOD_MS;
    for(;;)
    {
        HAL_GPIO_WritePin(GPIOD, LD5_Pin,GPIO_PIN_RESET);
        vTaskDelay(xDelay);
        HAL_GPIO_WritePin(GPIOD, LD5_Pin,GPIO_PIN_SET);
        vTaskDelay(xDelay);
    }
}
This task toggles LED LD5 on and off every second.

Led_GateKeeper3
c
void Led_GateKeeper3(void *pvParameters)
{
    TickType_t xDelay = 1000 / portTICK_PERIOD_MS;
    for(;;)
    {
        if (HAL_GPIO_ReadPin(GPIOA,B1_Pin)==1)
        {
            vTaskSuspend(myTask1Handle);
            vTaskSuspend(myTask2Handle);
            HAL_GPIO_WritePin(GPIOD, LD6_Pin,GPIO_PIN_SET);
            vTaskDelay(xDelay);
            HAL_GPIO_WritePin(GPIOD, LD6_Pin,GPIO_PIN_RESET);
            vTaskDelay(xDelay);
        }
        if (HAL_GPIO_ReadPin(GPIOA,B1_Pin)==0)
        {
            vTaskSuspend(myTask1Handle);
            vTaskSuspend(myTask2Handle);
            HAL_GPIO_WritePin(GPIOD, LD6_Pin,GPIO_PIN_RESET);
            vTaskDelay(xDelay);
        }
    }
}
This task checks the state of a button (B1). If the button is pressed, it suspends myTask1Handle and myTask2Handle, and toggles LED LD6. If the button is not pressed, it suspends the tasks and turns off LED LD6.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
5. Main Function
c
int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_I2C1_Init();
    MX_I2S3_Init();
    MX_SPI1_Init();

    xTaskCreate(Led_GateKeeper, "led_gate", configMINIMAL_STACK_SIZE, 0, 2, &myTask1Handle);
    xTaskCreate(Led_GateKeeper1, "led_gate", configMINIMAL_STACK_SIZE, 0, 2, &myTask2Handle);
    xTaskCreate(Led_GateKeeper3, "led_gate", configMINIMAL_STACK_SIZE, 0, 2, NULL);

    osThreadDef(defaultTask, StartDefaultTask, osPriorityNormal, 0, 128);
    defaultTaskHandle = osThreadCreate(osThread(defaultTask), NULL);

    osKernelStart();

    while (1)
    {
    }
}
The main function initializes the HAL library, configures the system clock, and initializes the peripherals. It then creates three tasks (Led_GateKeeper, Led_GateKeeper1, and Led_GateKeeper3) and starts the FreeRTOS scheduler.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
6. System Clock Configuration
c
void SystemClock_Config(void)
{
    RCC_OscInitTypeDef RCC_OscInitStruct = {0};
    RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
    __HAL_RCC_PWR_CLK_ENABLE();
    __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

    RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
    RCC_OscInitStruct.HSEState = RCC_HSE_ON;
    RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
    RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
    RCC_OscInitStruct.PLL.PLLM = 8;
    RCC_OscInitStruct.PLL.PLLN = 336;
    RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
    RCC_OscInitStruct.PLL.PLLQ = 7;
    if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
    {
        Error_Handler();
    }

    RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                                |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
    RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
    RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
    RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
    RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;

    if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_5) != HAL_OK)
    {
        Error_Handler();
    }
}
This function configures the system clock to use the High-Speed External (HSE) oscillator and sets up the PLL for the desired clock frequencies.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
7. Peripheral Initialization Functions
These functions initialize the I2C, I2S, SPI, and GPIO peripherals.

I2C Initialization
c
static void MX_I2C1_Init(void)
{
    hi2c1.Instance = I2C1;
    hi2c1.Init.ClockSpeed = 100000;
    hi2c1.Init.DutyCycle = I2C_DUTYCYCLE_2;
    hi2c1.Init.OwnAddress1 = 0;
    hi2c1.Init.AddressingMode = I2C_ADDRESSINGMODE_7BIT;
    hi2c1.Init.DualAddressMode = I2C_DUALADDRESS_DISABLE;
    hi2c1.Init.OwnAddress2 = 0;
    hi2c1.Init.GeneralCallMode = I2C_GENERALCALL_DISABLE;
    hi2c1.Init.NoStretchMode = I2C_NOSTRETCH_DISABLE;
    if (HAL_I2C_Init(&hi2c1) != HAL_OK)
    {
        Error_Handler();
    }
}
I2S Initialization
c
static void MX_I2S3_Init(void)
{
    hi2s3.Instance = SPI3;
    hi2s3.Init.Mode = I2S_MODE_MASTER_TX;
    hi2s3.Init.Standard = I2S_STANDARD_PHILIPS;
    hi2s3.Init.DataFormat = I2S_DATAFORMAT_16B;
    hi2s3.Init.MCLKOutput = I2S_MCLKOUTPUT_ENABLE;
    hi2s3.Init.AudioFreq = I2S_AUDIOFREQ_96K;
    hi2s3.Init.CPOL = I2S_CPOL_LOW;
    hi2s3.Init.ClockSource = I2S_CLOCK_PLL;
    hi2s3.Init.FullDuplexMode = I2S_FULLDUPLEXMODE_DISABLE;
    if (HAL_I2S_Init(&hi2s3) != HAL_OK)
    {
        Error_Handler();
    }
}
SPI Initialization
c
static void MX_SPI1_Init(void)
{
    hspi1.Instance = SPI1;
    hspi1.Init.Mode = SPI_MODE_MASTER;
    hspi1.Init.Direction = SPI_DIRECTION_2LINES;
    hspi1.Init.DataSize = SPI_DATASIZE_8BIT;
    hspi1.Init.CLKPolarity = SPI_POLARITY_LOW;
    hspi1.Init.CLKPhase = SPI_PHASE_1EDGE;
    hspi1.Init.NSS = SPI_NSS_SOFT;
    }

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
    for further questions mail me on : dagmazen@gmail.com
