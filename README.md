# STM32 FreeRTOS Mutex Example

## Overview
This project demonstrates mutual exclusion (mutex) usage in an STM32 microcontroller with FreeRTOS. Mutexes are used to protect shared resources (UART in this case) from concurrent access by multiple tasks, ensuring data integrity and preventing output corruption.

> This project uses native FreeRTOS APIs directly, not the CMSIS-RTOS wrapper.

## Project Description
The application creates three tasks with different priorities:
- HPT_Task (Highest Priority: 3) - Attempts to access UART every 750ms
- MPT_Task (Medium Priority: 2) - Simple task that prints periodically (no mutex)
- LPT_Task (Lowest Priority: 1) - Attempts to access UART every 1000ms

A mutex protects the UART transmission function, demonstrating how mutual exclusion ensures that only one task at a time can access the shared resource. The example also shows priority inheritance, a key feature of mutexes that prevents priority inversion.

## Key Concept: Mutual Exclusion
- Shared Resource Protection: UART is protected by a mutex
- Priority Inheritance: Prevents priority inversion when lower priority task holds mutex
- Atomic Access: Ensures complete UART messages are transmitted without interruption
- Task Synchronization: Tasks block when mutex is already held

## UART Output

https://github.com/user-attachments/assets/046c9d5c-74e3-4b9a-87ab-0f638013cfd7

## Hardware Requirements
- STM32 development board (STM32F103C8T6 "Blue Pill")
- USB-to-UART converter (for viewing debug messages)
- UART connection (PA9/PA10 for USART1 on STM32F103)

## Pin Configuration
| Pin | Function | Description | 
|-----|----------|-------------|
| PA9 | USART1 TX | Debug output from STM32 |
| PA10 | USART1 RX | (Reserved for future use) |

## Task Priorities and Behavior
| Task | Priority | Action | Delay | Mutex Usage |
|------|----------|--------|-------|-------------|
| HPT_Task | 3 (Highest) | Transmits "In HPT =====" | 750ms | mutex (deliberate 5 sec delay) |
| MPT_Task | 2 | Transmits "In MPT =====" | 2000ms | No mutex (independent output) |
| LPT_Task | 1 (Lowest) | Transmits "In LPT =====" | 1000ms | Takes mutex (deliberate 5 sec delay) |

## Mutex Configuration
- Type: Binary Mutex (xSemaphoreCreateMutex)
- Initial State: Available (unlocked)
- Features: Priority inheritance enabled
- Blocking: Tasks block indefinitely (portMAX_DELAY) when mutex is held

## Special Feature: Simulated Long Operation
The `Send_Uart()` function includes a `HAL_Delay(5000)` to simulate a long operation (e.g., sensor reading, data processing) while holding the mutex. This demonstrates:

- Long Critical Sections: How mutexes protect resources for extended periods
- Task Blocking: Lower priority tasks waiting for mutex
- Priority Inversion: Why priority inheritance is crucial

Key Concepts
1. Mutual Exclusion (Mutex)
    - Only one task can execute the protected UART transmission at a time
    - Other tasks block until mutex is released
    - Ensures complete messages without corruption
2. Priority Inheritance<br>
    When HPT_Task (priority 3) wants the mutex held by LPT_Task (priority 1):
    - LPT_Task inherits priority 3 temporarily
    - Prevents MPT_Task (priority 2) from preempting
    - LPT_Task completes faster, releases mutex, returns to priority 1
    - Priority inversion is avoided

3. Task Blocking
    - Tasks block automatically when mutex is unavailable
    - Scheduler runs other ready tasks
    - Tasks unblock when mutex is released

4. Critical Section Protection
    - UART operations are atomic (cannot be interrupted)
    - Data integrity maintained
    - No message interleaving

## Mutex vs Binary Semaphore Comparison

| Feature | Mutex | Binary Semaphore | 
|---------|-------|------------------|
| Purpose | Mutual exclusion | Synchronization |
| Priority Inheritance | Yes | No |
| Initial State | Available (1) | Usually unavailable (0) |
| Who can give | Only the task that took it | Any task/ISR |
| Recursive | Protect shared resources | Signal events |
| Priority Inheritance | Can be recursive | Not recursive |

## Priority Inversion Scenario
Without priority inheritance:
1. LPT_Task (priority 1) takes mutex
2. HPT_Task (priority 3) preempts and tries to take mutex → blocks
3. MPT_Task (priority 2) preempts LPT_Task (since priority 2 > 1)
4. HPT_Task is blocked indefinitely behind MPT_Task

With priority inheritance (how mutex works):
1. LPT_Task (priority 1) takes mutex
2. HPT_Task (priority 3) tries to take mutex → blocks
3. LPT_Task inherits priority 3 temporarily
4. MPT_Task (priority 2) cannot preempt LPT_Task
5. LPT_Task releases mutex, returns to priority 1
6 .HPT_Task gets mutex and runs


## Contact
**Rubin Khadka Chhetri**  
📧 rubinkhadka84@gmail.com <br>
🐙 GitHub: https://github.com/rubin-khadka
