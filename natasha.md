* The handler then searches for the service:

  ```c
  case ESP_GATTC_CFG_MTU_EVT:
        if (param->cfg_mtu.status != ESP_GATT_OK){
            ESP_LOGE(GATTC_TAG,"Config mtu failed");
        }
        ESP_LOGI(GATTC_TAG, "Status %d, MTU %d, conn_id %d", param->cfg_mtu.status, param->cfg_mtu.mtu, param->cfg_mtu.conn_id);
        esp_ble_gattc_search_service(gattc_if, param->cfg_mtu.conn_id, &remote_filter_service_uuid);
        break;
        ```
If the service is found, an ``ESP_GATTC_SEARCH_RES_EVT`` event is triggered which allows to set the ``get_service_1 flag`` to true. This flag is used to print information and later get the characteristic that the client is interested in.


* The handler then searches for the service:

  ```c
  case ESP_GATTC_CFG_MTU_EVT:
        if (param->cfg_mtu.status != ESP_GATT_OK){
            ESP_LOGE(GATTC_TAG,"Config mtu failed");
        }
        ESP_LOGI(GATTC_TAG, "Status %d, MTU %d, conn_id %d", param->cfg_mtu.status, param->cfg_mtu.mtu, param->cfg_mtu.conn_id);
        esp_ble_gattc_search_service(gattc_if, param->cfg_mtu.conn_id, &remote_filter_service_uuid);
        break;
  ```
  If the service is found, an ``ESP_GATTC_SEARCH_RES_EVT`` event is triggered which allows to set the ``get_service_1 flag`` to true. This flag is used to print information and later get the characteristic that the client is interested in.
  
  
  * The handler then searches for the service:

  ```c
  case ESP_GATTC_CFG_MTU_EVT:
        if (param->cfg_mtu.status != ESP_GATT_OK){
            ESP_LOGE(GATTC_TAG,"Config mtu failed");
        }
        ESP_LOGI(GATTC_TAG, "Status %d, MTU %d, conn_id %d", param->cfg_mtu.status, param->cfg_mtu.mtu, param->cfg_mtu.conn_id);
        esp_ble_gattc_search_service(gattc_if, param->cfg_mtu.conn_id, &remote_filter_service_uuid);
        break;
  ```
If the service is found, an ``ESP_GATTC_SEARCH_RES_EVT`` event is triggered which allows to set the ``get_service_1 flag`` to true. This flag is used to print information and later get the characteristic that the client is interested in.


  * The handler then searches for the service:

  ```c
  case ESP_GATTC_CFG_MTU_EVT:
        if (param->cfg_mtu.status != ESP_GATT_OK){
            ESP_LOGE(GATTC_TAG,"Config mtu failed");
        }
        ESP_LOGI(GATTC_TAG, "Status %d, MTU %d, conn_id %d", param->cfg_mtu.status, param->cfg_mtu.mtu, param->cfg_mtu.conn_id);
        esp_ble_gattc_search_service(gattc_if, param->cfg_mtu.conn_id, &remote_filter_service_uuid);
        break;
  ```

	If the service is found, an ``ESP_GATTC_SEARCH_RES_EVT`` event is triggered which allows to set the ``get_service_1 flag`` to true. This flag is used to print information and later get the characteristic that the client is interested in.