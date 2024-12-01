# Repo1
/* USER CODE BEGIN Header */
/**
 ******************************************************************************
 * @file           : main.c
 * @brief          : Main program body
 ******************************************************************************
 * @attention
 *
 * Copyright (c) 2022 STMicroelectronics.
 * All rights reserved.
 *
 * This software is licensed under terms that can be found in the LICENSE file
 * in the root directory of this software component.
 * If no LICENSE file comes with this software, it is provided AS-IS.
 *
 ******************************************************************************
 */
/* USER CODE END Header */
/* Includes ------------------------------------------------------------------*/
#include "main.h"
#include "dma.h"
#include "tim.h"
#include "usart.h"
#include "gpio.h"

/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include "stdio.h"
/* USER CODE END Includes */

/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */

/* USER CODE END PTD */

/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
/* USER CODE END PD */

/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */

/* USER CODE END PM */

/* Private variables ---------------------------------------------------------*/

/* USER CODE BEGIN PV */
uint8_t count = 0;					//к-ть нати�?кать
uint8_t btn_cur = 0;				//поточне значенн�? кнопки
uint8_t btn_prev = 0; 				//попереднє значенн�? кнопки
uint8_t active_brightness = 50;		//активна �?�?краві�?ть
uint8_t off_brightness = 0;			//вимкнена �?�?краві�?ть

volatile uint8_t counter = 0;		//рахівник на колбеці У�?РТ
volatile uint8_t buff[16] = { 0, };   //буфер дл�? збереженн�? введеної інформації value
volatile uint8_t value = 0;	//змінна в котру запи�?ують байт інформаціїї з терміналу
volatile uint8_t flag = 0; //флаг дл від�?лідковуванн�? введенн�? команд в термінал

int str[255] = { 0, };					//ма�?ив де зберігають�?�? введені дані з ьерміналу
/* USER CODE END PV */

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
/* USER CODE BEGIN PFP */

/* USER CODE END PFP */

/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */

/* USER CODE END 0 */

/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
  /* USER CODE BEGIN 1 */

  /* USER CODE END 1 */

  /* MCU Configuration--------------------------------------------------------*/

  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();

  /* USER CODE BEGIN Init */

  /* USER CODE END Init */

  /* Configure the system clock */
  SystemClock_Config();

  /* USER CODE BEGIN SysInit */

  /* USER CODE END SysInit */

  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_DMA_Init();
  MX_TIM1_Init();
  MX_TIM2_Init();
  MX_USART3_UART_Init();
  /* USER CODE BEGIN 2 */

	setvbuf(stdin, NULL, _IONBF, 0); // определение нулевого буфера

	HAL_UART_Receive_IT(&huart3, &value, 1); //запу�?каем UART по прерыванию

	HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_1);
	HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_2);

	printf("PRINT BRIGHTNESS FOR LED In range from 0 to 100->>>>!\r\n");
  /* USER CODE END 2 */

  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
	while (1)
	{
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
		if (flag == 1)
		{
			sscanf(&buff[0], "%d", &str[0]);
			if ((str[0] > 100) || (str[0] < 0))
			{
				printf("Error. Please enter correct value!\r\n");
			}
			else
			{
				active_brightness = str[0];
				printf("You enter %d value\r\n", active_brightness);
				printf(
						"PRINT BRIGHTNESS FOR LED In range from 0 to 100->>>>!\r\n");
			}
			flag = 0;
		}

		btn_cur = HAL_GPIO_ReadPin(BUTTON_EXTI_GPIO_Port, BUTTON_EXTI_Pin);
		switch (count)
		{
		case 0:
		{
			TIM2->CCR1 = off_brightness;
			TIM2->CCR2 = off_brightness;
		}
			break;
		case 1:
		{
			TIM2->CCR1 = active_brightness;
			TIM2->CCR2 = off_brightness;

		}
			break;
		case 2:
		{
			TIM2->CCR1 = off_brightness;
			TIM2->CCR2 = active_brightness;
		}
			break;
		case 3:
		{
			TIM2->CCR1 = active_brightness;
			TIM2->CCR2 = active_brightness;
		}
			break;
		}
		HAL_Delay(5);
		btn_prev = btn_cur;
	}
  /* USER CODE END 3 */
}

/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Configure the main internal regulator output voltage
  */
  __HAL_RCC_PWR_CLK_ENABLE();
  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = RCC_HSICALIBRATION_DEFAULT;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSI;
  RCC_OscInitStruct.PLL.PLLM = 8;
  RCC_OscInitStruct.PLL.PLLN = 180;
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
  RCC_OscInitStruct.PLL.PLLQ = 4;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  /** Activate the Over-Drive mode
  */
  if (HAL_PWREx_EnableOverDrive() != HAL_OK)
  {
    Error_Handler();
  }

  /** Initializes the CPU, AHB and APB buses clocks
  */
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

/* USER CODE BEGIN 4 */

/* USER CODE END 4 */

/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
	/* User can add his own implementation to report the HAL error return state */
	__disable_irq();
	while (1)
	{
	}
  /* USER CODE END Error_Handler_Debug */
}

#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None

  */

  /*
 * exti_func.c
 *
 *  Created on: Oct 8, 2022
 *      Author: shoko
 */


#include "main.h"
extern uint8_t flag;
extern uint8_t btn_cur ;
extern uint8_t btn_prev ;
extern uint8_t count;

extern TIM_HandleTypeDef htim1;


void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{

	if(htim->Instance == TIM1)
	{
		HAL_TIM_Base_Stop_IT(&htim1); // останавливаем таймер
		__HAL_GPIO_EXTI_CLEAR_IT(BUTTON_EXTI_Pin);  // очищаем бит EXTI_PR (бит прерывания)
		NVIC_ClearPendingIRQ(EXTI15_10_IRQn); // очищаем бит NVIC_ICPRx (бит очереди)
		HAL_NVIC_EnableIRQ(EXTI15_10_IRQn);   // включаем внешнее прерывание
		btn_cur = HAL_GPIO_ReadPin(BUTTON_EXTI_GPIO_Port, BUTTON_EXTI_Pin);
		if((btn_prev == 0) && (btn_cur != 0))
		{
			count++;
			if(count == 4)
			{
				count = 0;
			}
		}
	}
}

void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
	if(GPIO_Pin == BUTTON_EXTI_Pin)
	{
		HAL_NVIC_DisableIRQ(EXTI15_10_IRQn); // сразу же отключаем прерывания на этом пине
		// либо выполняем какое-то действие прямо тут, либо поднимаем флажок
		HAL_TIM_Base_Start_IT(&htim1); // запускаем таймер
	}
}

void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
	/* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */

/*
 * usart3.c
 *
 *  Created on: 9 окт. 2022 г.
 *      Author: shoko
 */

#include "main.h"
#include "stdio.h"

extern volatile uint8_t counter;
extern volatile uint8_t buff[16];
extern volatile uint8_t value;
extern volatile uint8_t flag;

extern int str[255];

extern UART_HandleTypeDef huart3;
extern DMA_HandleTypeDef hdma_usart3_tx;
extern DMA_HandleTypeDef hdma_usart3_rx;

int __io_putchar(int ch)
{
	HAL_UART_Transmit(&huart3, (uint8_t*) &ch, 1, HAL_MAX_DELAY);
	return 0;
}
//Receive one char in blocking mode
int __io_getchar(void)
{
	uint8_t result;
	__HAL_UART_CLEAR_OREFLAG(&huart3);
	HAL_UART_Receive(&huart3, &result, 1, HAL_MAX_DELAY);
	if (result == '\r')
		result = '\n'; // Dirty hack. Replace "return" character with "new line" character
	return (int) result;
}

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart == &huart3)
	{
		HAL_UART_Receive_IT(&huart3, &value, 1);
		//	sscanf(value,"%d",&buff[counter]);
		buff[counter] = value;

		if (value == '\r')
		{
			counter = 0;
			flag = 1;
		}
		else
			counter++;
	}
}
