```python
from rest_framework import status
from rest_framework.response import Response
from rest_framework import serializers
# from core.api.message.dispatcher.utils import Dispatcher
from django.core.cache import cache
from django.http import HttpResponse
from rest_framework.exceptions import APIException
from rest_framework.exceptions import ValidationError
from core.utils.process_api import ApiFulfilment
# from core.utils.misc import get_channel_object
from channels_interface.controllers.utils import get_channel_object
from channels_interface.controllers.dispatcher import Dispatcher
import logging,json
import asyncio

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

class NotifySerializer(serializers.Serializer):
    tenant = serializers.CharField(max_length=50)
    utter_name = serializers.CharField(max_length=100)
    # mobile_number = serializers.CharField(max_length=200)
    channel_id = serializers.CharField(max_length=100)
    channel_code = serializers.CharField(max_length=100)



def callback(request, **kwargs):
    logger.debug(f"params: {request.query_params}")
    logger.debug(f"kwargs: {kwargs}")
    channel_id = request.query_params["channel_id"]
    channel_code = request.query_params["channel_code"]  
    logger.debug(f"channel_id: {channel_id} ------- channel_code: {channel_code}")

    serializer = NotifySerializer(
        data=dict(
            tenant=kwargs["tenant"],
            channel_id=channel_id,
            channel_code=channel_code,
            utter_name=request.query_params["utter_show_payment_success"], 
        )
    )
    serializer.is_valid(raise_exception=True)
    notify_user(**serializer.validated_data)
    return close_page()



def notify_user(**kwargs):
    try:
        dispatcher = Dispatcher(
            get_channel_object(
                kwargs["channel_code"], kwargs["channel_id"], kwargs["tenant"]
            )
        )
        asyncio.run(dispatcher.utter_template(kwargs["utter_name"]))
    except Exception as e:
        raise ValidationError("utter retri failure...")

def close_page(bbc_cookie=None,bbp_cookie=None,bbetb_cookie=None):    
    return Response(
                {"message": "Message sent user successfully"},
                status=status.HTTP_200_OK
            )



#test links
# https://beta1.studio.bankbuddy.me/bot/callback/insurebuddyone/submit/?channel_id={channel_id}&channel_code={channel_code}&tenant={tenant}&utter_show_payment_success=utter_show_payment_success

```
