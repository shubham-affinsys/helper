
### Validator Example
```python
#validator
from core.utils import validators
from django.core.exceptions import ValidationError
# from ..utils.params_config import MAX_RETRIES_LIMIT
import logging
import re
MAX_RETIRES_LIMIT=3
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

RMN_REGEX =  r"(?P<main>(^[\+]?[(]?[0-9]{3}[)]?[-\s\.]?[0-9]{3}[-\s\.]?[0-9]{4,6}$))"

class MobileValidator(validators.BaseValidator):
  async def __call__(self, *args, **kwargs):
    # Logging validator start
    logging.debug("Mobile no validator triggered...")
    user_input = self.value
    logger.debug(f"User input: {user_input}")

    # Get user intent
    current_intent = self.value
    logger.debug(f"user intent ================> {current_intent}")

    # Initialize slot variables
    slot_abort = "no"
    slot_rmn = ""

    # Handle User cancel input
    if current_intent == "cancel":
        logger.debug("user cancelled...")
        # Log Transcation stage
        self.slots["slot_final_log_to_producer"] = self.log_stage("Final Response","User Aborted","User Cancelled")
        await self.validation_failure(flow_end_utter = "utter_cancelled",flow_end=True)
        slot_abort = "yes"
    else:
        logger.debug(f"if case User input: {user_input}")
        # Extract user data using regex
        regex_pattern = RMN_REGEX
        if len(re.findall(regex_pattern,user_input))==1:
            slot_rmn = re.search(regex_pattern,user_input).group("main")
            logger.debug(f"slot_rmn: {slot_rmn}")
            slot_abort = "no"
            logger.debug(f"User input: {user_input}")
        # No valid data found . procced with retry
        else:
            logger.debug(f"else case User input: {user_input}")
            slot_abort = "yes"
            slot_rmn = ""
            await self.validation_failure(
                max_retries = MAX_RETIRES_LIMIT,
                validation_failure_utter = "utter_invalid_rmn",
                max_retries_reached_utter = "utter_max_retries",
            )
            logging.error("validation faliure")
        # Store the slot variables
        logger.debug(f"final User input: {user_input}")
        self.slots["slot_abort"] = slot_abort
        self.slots["slot_rmn"] = slot_rmn
```