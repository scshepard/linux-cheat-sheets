# Acceso SSH sin contraseña

Ref <http://www.gentoo.org/doc/en/keychain-guide.xml> <http://help.github.com/ssh-key-passphrases/>

## Generar las claves

	ssh-keygen -t dsa
	ssh-keygen -t rsa

Comprobar los permisos

	ls ~/.ssh/id_*
	-rw------- id_dsa
	-rw-r--r-- id_dsa.pub
	-rw------- id_rsa
	-rw-r--r-- id_rsa.pub

## Copiar las claves

	ssh-copy-id -i ~/.ssh/id_dsa.pub usuario@servidor

Para indicar el puerto

	ssh-copy-id -i ~/.ssh/id_dsa.pub "usuario@servidor -p 65535" # Si no funciona probar sin las comillas

## SSH-Agent

El programa ssh-agent sirve para descifrar nuestra(s) clave(s) privada una sola vez y así poder usar ssh libremente sin más contraseñas.

Añadir a ~/.bashrc

	SSH_ENV="$HOME/.ssh/environment"

	# start the ssh-agent
	function start_agent {
		echo "Initializing new SSH agent..."
		# spawn ssh-agent
		ssh-agent | sed 's/^echo/#echo/' > "$SSH_ENV"
		echo succeeded
		chmod 600 "$SSH_ENV"
		. "$SSH_ENV" > /dev/null
		ssh-add
	}

	# test for identities
	function test_identities {
		# test whether standard identities have been added to the agent already
		ssh-add -l | grep "The agent has no identities" > /dev/null
		if [ $? -eq 0 ]; then
			ssh-add
			# $SSH_AUTH_SOCK broken so we start a new proper agent
			if [ $? -eq 2 ];then
				start_agent
			fi
		fi
	}

	# check for running ssh-agent with proper $SSH_AGENT_PID
	if [ -n "$SSH_AGENT_PID" ]; then
		ps -ef | grep "$SSH_AGENT_PID" | grep ssh-agent > /dev/null
		if [ $? -eq 0 ]; then
		test_identities
		fi
	# if $SSH_AGENT_PID is not properly set, we might be able to load one from
	# $SSH_ENV
	else
		if [ -f "$SSH_ENV" ]; then
		. "$SSH_ENV" > /dev/null
		fi
		ps -ef | grep "$SSH_AGENT_PID" | grep -v grep | grep ssh-agent > /dev/null
		if [ $? -eq 0 ]; then
			test_identities
		else
			start_agent
		fi
	fi

Para integrar ssh-agent en KDE, edita /etc/kde/startup/agent-startup.sh y /etc/kde/shutdown/agent-shutdown.sh y descomenta las líneas que hacen referencia a ssh-agent.

	# /etc/kde/startup/agent-startup.sh
	if [ -x /usr/bin/ssh-agent ]; then
	  eval "$(/usr/bin/ssh-agent -s)"
	fi

	#/etc/kde/shutdown/agent-shutdown.sh

	if [ -n "${SSH_AGENT_PID}" ]; then
	  eval "$(ssh-agent -k)"
	fi

Tras esos pasos basta con ejecutar `ssh-add` para que KDE recuerde nuestra contraseña durante toda la sesión.
