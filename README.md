# Atividade RabbitMQ - Work Queues

Este script é a atividade prática do exemplo Work Queues do RabbitMQ usando Python.
O objetivo é enviar mensagens para uma fila e processá-las em sequência com workers simples.

Ferramentas utilizadas:
- Python 3
- RabbitMQ
- Biblioteca pika
- IDE: PyCharm

Como executar:
1. Instale a dependência:
   pip install pika
2. Inicie o servidor RabbitMQ (certifique-se de que está rodando).
3. Execute este script:
   python work_queues.py
O script enviará mensagens para a fila e as processará em seguida.

Resultado esperado:
[x] Enviada: Tarefa 1.
[x] Enviada: Tarefa 2..
[x] Enviada: Tarefa 3...
Todas as mensagens foram enviadas.

Aguardando mensagens. Pressione Ctrl+C para sair.
[x] Recebida: Tarefa 1.
[x] Concluída
[x] Recebida: Tarefa 2..
[x] Concluída
[x] Recebida: Tarefa 3...
[x] Concluída

Resumo:
- O script envia mensagens para a fila 'task_queue' e as processa.
- Cada ponto '.' nas mensagens simula 1 segundo de processamento.
- Tudo feito no PyCharm, com RabbitMQ rodando localmente e usando a biblioteca pika.
"""

import pika
import time

## Conexão com RabbitMQ
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

## Cria fila durável
channel.queue_declare(queue='task_queue', durable=True)

## Mensagens para enviar
messages = ["Tarefa 1.", "Tarefa 2..", "Tarefa 3..."]

## Envia mensagens
for msg in messages:
    channel.basic_publish(
        exchange='',
        routing_key='task_queue',
        body=msg,
        properties=pika.BasicProperties(delivery_mode=2)
    )
    print(f"[x] Enviada: {msg}")

print("Todas as mensagens foram enviadas.\n")

## Processa mensagens da fila
def callback(ch, method, properties, body):
    print(f"[x] Recebida: {body.decode()}")
    time.sleep(body.count(b'.'))  # Simula tempo de processamento
    print("[x] Concluída")
    ch.basic_ack(delivery_tag=method.delivery_tag)

# Configura o consumo da fila
channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='task_queue', on_message_callback=callback)

print("Aguardando mensagens. Pressione Ctrl+C para sair.")
channel.start_consuming()
