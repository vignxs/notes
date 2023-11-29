# Create a custom format that includes elapsed time
def custom_format(record):
    current_time = time.time()
    elapsed_time = current_time - getattr(record, "start_time", current_time)
    record.start_time = current_time
    return "<green>{time:HH:mm:ss}</green> | <level>{level: <8}</level> | <level>{message}</level> | <level>Elapsed Time: {elapsed_time:.2f} seconds</level>"

# Configure Loguru logger with custom format
logger.remove()  # Remove the default sink
logger.add(sys.stderr, format=custom_format, colorize=True)  # Log to console
logger.add("my_log.log", format=custom_format)  # Log to a file
