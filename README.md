# Desafio Debug
O código implementa um sistema de execução de tarefas assíncronas com um worker thread que consome tarefas de uma fila (Queue) enquanto a aplicação está ativa. A arquitetura base consiste em:
- Fila de tarefas (Queue): onde as tarefas são enfileiradas para processamento.
- Worker thread: que consome tarefas da fila enquanto a aplicação estiver rodando ou houver tarefas pendentes.
- Métodos de controle (start() / stop()): que inicializam e finalizam a execução do worker.

No entanto, a implementação atual contém problemas propositais, que foram inseridos para destacar pontos que exigem atenção e refinamento em um código de produção. Identifique esses pontos abaixo:

<pre>
  <code>
    import threading
    import time
    import random
    from queue import Queue
    
    class TaskProcessor:
        def __init__(self, num_workers: int = 4):
            self.task_queue = Queue()
            self.num_workers = num_workers
            self.workers = []
            self.results = []
            self.lock = threading.Lock()
            self.running = False
            
        def add_task(self, task):
            """Adiciona uma tarefa à fila"""
            with self.lock:
                self.task_queue.put(task)
        
        def worker(self):
            """Função executada por cada thread worker"""
            while self.running or not self.task_queue.empty():
                try:
                    task = self.task_queue.get(timeout=0.1)
                    
                    processing_time = random.uniform(0.1, 0.5)
                    time.sleep(processing_time)
                    
                    result = f"Processed task {task} in {processing_time:.2f}s"
                    
                    with self.lock:
                        self.results.append(result)
                    
                    self.task_queue.task_done()
                except Exception as e:
                    continue
        
        def start(self):
            """Inicia os workers"""
            self.running = True
            self.workers = [
                threading.Thread(target=self.worker, daemon=True)
                for _ in range(self.num_workers)
            ]
            
            for worker in self.workers:
                worker.start()
        
        def stop(self):
            """Para os workers"""
            self.running = False
            for worker in self.workers:
                worker.join(timeout=1.0)
        
        def get_results(self):
            """Retorna os resultados processados"""
            with self.lock:
                return self.results.copy()
    
    def main():
        processor = TaskProcessor(num_workers=4)
        
        for i in range(20):
            processor.add_task(f"Task-{i+1}")
        
        processor.start()
        
        for i in range(20, 30):
            time.sleep(random.uniform(0.05, 0.2))
            processor.add_task(f"Task-{i+1}")
        
        time.sleep(2)
    
        processor.stop()
        
        results = processor.get_results()
        print("\n".join(results))
        print(f"\nTotal tasks processed: {len(results)}")
    
    if __name__ == "__main__":
        main()
  </code>
</pre>
