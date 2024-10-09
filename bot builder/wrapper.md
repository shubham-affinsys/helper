```python
#wrapper
from core.utils import wrappers
import logging
logger = logging.getLogger(__name__)
import requests

class Wrapper(wrappers.BaseWrapper):
  async def __call__(self, *args, **kwargs):
    logger.debug("Entering wrapper....accounts")

    # fetch all acoounts and store it in a account list of dictionaries
    response = self.call_api(api_name="API_EXT_GET_ACCOUNTS")
    data = response.json()
    
    # format data recieved in form of list and dictionaries to use it as quick replies
    accounts=[]
    for acc in data["accounts"]:
        account = dict()
        account["title"] = acc
        account["payload"] = acc
        account["balance"] = data["accounts"][acc]["balance"]
        #adding account to list
        accounts.append(account)

    self.slots["slot_accounts"] =  accounts
```
