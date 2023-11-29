from loguru import logger
import time

# Create a custom format that includes elapsed time
def custom_format(record):
    if "start_time" not in record.thread:
        record.thread.start_time = time.time()
    
    current_time = time.time()
    elapsed_time = current_time - record.thread.start_time
    return f"{record.time:HH:mm:ss} | {record.level: <8} | {record.message} | Elapsed Time: {elapsed_time:.2f} seconds"

# Configure Loguru logger with custom format
logger.remove()  # Remove the default sink
logger.add(sys.stderr, format=custom_format, colorize=True)  # Log to console
logger.add("my_log.log", format=custom_format)  # Log to a file

# Log some messages
logger.info("First logger message")
time.sleep(2)  # Simulate a delay
logger.info("Second logger message")
