keycloak:
	docker compose -f docker-compose.keycloak.yml up -d
kong-postgres:
	COMPOSE_PROFILES=database KONG_DATABASE=postgres docker compose up -d

kong-dbless:
	docker-compose up -d

clean:
	docker compose kill
	docker compose rm -f -v

# Go to http://localhost:8085/auth/admin/thg/console/#/thg/clients then create a client with id kong then enable "Client authentication", "Authorization"
# then go to Credentials tab to copy Client Secret and replace it in the below kong_secret var
kong_secret=3UURHJ5yaU2OhLZYECDPWS4sPVmfSFl8
init: create_service create_route configure_oidc_plugin

create_service:
	curl -s -X POST http://localhost:8001/services \
			-d name=mock-request \
			-d url=http://mockbin.org/request
	curl -s -X POST http://localhost:8001/services \
			-d name=userinfo \
			-d url=http://keycloak:8080/auth/realms/thg/protocol/openid-connect/userinfo

create_route:
	curl -i -X POST http://localhost:8001/services/mock-request/routes \
		  --data 'paths[]=/mock' \
		  --data name=mock-request

	curl -i -X POST http://localhost:8001/services/userinfo/routes \
		  --data 'paths[]=/userinfo' \
		  --data name=userinfo

# It will configure the plugin with the Kong client created in Keycloak
# Remember to replace your keycloak server IP
configure_oidc_plugin:
	curl -s -X POST http://localhost:8001/plugins \
          -d name=oidc \
          -d config.client_id=client-portal \
          -d config.client_secret=${kong_secret} \
          -d config.discovery=http://172.30.188.87:8085/auth/realms/thg/.well-known/openid-configuration \
          -d config.introspection_endpoint=http://172.30.188.87:8085/auth/realms/thg/protocol/openid-connect/token/introspect \

verify_active_token:
	curl -s -X POST http://localhost:8085/auth/realms/thg/protocol/openid-connect/token/introspect \
		 -d token=<access-token> \
		 -d client_id=kong \
		 -d client_secret=${kong_secret} \
		 | python -mjson.tool


# It will configure the plugin with the Kong client created in Keycloak
# Remember to replace your keycloak server IP
patch_oidc_plugin:
	curl -s -X PATCH http://localhost:8001/plugins/ab75b1b6-5f8c-4a14-b1a2-ed2a9689567d \
          -d name=oidc \
          -d config.client_id=client-portal \
          -d config.discovery=http://172.25.134.64:8085/auth/realms/thg/.well-known/openid-configuration \
          -d config.introspection_endpoint=http://172.25.134.64:8085/auth/realms/thg/protocol/openid-connect/token/introspect \
		  -d config.access_token_as_bearer="yes" \
		  -d config.access_token_header_name="Authorization" 
		#   -d config.scope="openid profile email roles" \
		#   -d config.validate_scope="yes" \
		#   -d config.bearer_only=true \
		#   -d config.bearer_jwt_auth_enable=true \
        #   -d config.introspection_cache_ignore=false \
        #   -d config.session_storage=redis \
        #   -d config.session_redis_host=redis \
        #   -d config.session_redis_port=6379
