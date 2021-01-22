---
title: stm32cubemx踩坑记
date: 2019-04-16 09:10:19
tags: [单片机]
categories: [Debug]
---

# spi

## spi的16位数据接收模式下size的问题

spi发送有8位和16位发送/接收模式，但是HAL库中的接口只有一种。比如最基础的

```cpp
    HAL_StatusTypeDef HAL_SPI_Transmit(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout);
    HAL_StatusTypeDef HAL_SPI_Receive(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout);
```

在8位模式下指针和size没有问题，但是16位模式下指针还是uint_8 *。根据源码

```cpp
    HAL_StatusTypeDef HAL_SPI_Transmit(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout)
    {

        ......

        /* Set the transaction information */
        hspi->State = HAL_SPI_STATE_BUSY_TX;
        hspi->ErrorCode = HAL_SPI_ERROR_NONE;
        hspi->pTxBuffPtr = (uint8_t *)pData;
        hspi->TxXferSize = Size;
        hspi->TxXferCount = Size;

        ......

        /* Transmit data in 16 Bit mode */
        if (hspi->Init.DataSize == SPI_DATASIZE_16BIT)
        {
            ......
            /* Transmit data in 16 Bit mode */
            while (hspi->TxXferCount > 0U)
            {
                /* Wait until TXE flag is set to send data */
                if (__HAL_SPI_GET_FLAG(hspi, SPI_FLAG_TXE))
                {
                    hspi->Instance->DR = *((uint16_t *)hspi->pTxBuffPtr);
                    hspi->pTxBuffPtr += sizeof(uint16_t);
                    hspi->TxXferCount--;
                }
                ......
            }
        }
        /* Transmit data in 8 Bit mode */
        else
        {
            ......
            while (hspi->TxXferCount > 0U)
            {
                /* Wait until TXE flag is set to send data */
                if (__HAL_SPI_GET_FLAG(hspi, SPI_FLAG_TXE))
                {
                    *((__IO uint8_t *)&hspi->Instance->DR) = (*hspi->pTxBuffPtr);
                    hspi->pTxBuffPtr += sizeof(uint8_t);
                    hspi->TxXferCount--;
                }
                ......
            }
        }

        ......

    }
```

- 可以看出16位模式下调用这个函数，指针用的是同一个。所以需要强行转换为`uint_8 *`
- size的递减没有考虑字节的问题，所以输入的size是发送的16位数据的个数，并不是整个数组的大小。