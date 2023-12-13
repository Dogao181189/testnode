?!/usr/bin/env bash
set -e
- Obter o ambiente / OSO
environment?$(uname) (em inglês)

Função para sair com uma mensagem de erro
exit_with_error() ?
    echo "Erro: $1"
    saída 1
) ) Em (em,

Verifique o sistema operacional e obtenha as informações do processador
caso "$ambiente" em
    Linux (em inglês)
        processador $ (nome - m)
        ;
    Darwin) (em inglês)
        processador $ (nome - m)
        ;
    ?MINGW?)
        exit_with_error "$environment (Windows) ambiente ainda não suportado. Por favor, use WSL (WSL2 recomendado) ou uma VM Linux. Saída do instalador."
        ;
    ?)
        processador "Desconhecido"
        ;
Esac (em inglês)

Verifique se há processador ARM ou Desconhecido e sai se for verdadeiro, o que significa que o instalador não é suportado pelo processador
se [[ "$processador" ? ?"arm" ||| "$processador" ? "Unknown" ]]; então
    exit_with_error "processador ainda não suportado. Saída do instalador."
fi

Imprimir o ambiente detectado e o processador
eco "Ambiente $ ambiental com $processor encontrado."


Verifique se algum comando de hashing está disponível
se ! (command -v openssl ? /dev/null || comando -v shasum ? /dev/null || comando -v sha256sum ? /dev/null); então
  eco "Não encontrado comandos de hashing suportados."
  ler -p "Você gostaria de instalar openssl? (y/n) "-n 1 -r
  Echo

  se [[ $REPLY ? ?[Yy]$ ]]; em seguida,
    Detectar o gerenciador de pacotes e instalar o openssl
    se comando -v apt-get - /dev/null; então
      sudo apt-get update & etc. apt-get install -y openssl
    comando elif -v yum - /dev/null; então
      sudo yum install -y openssl
    comando elif -v dnf - /dev/null; em seguida,
      sudo dnf install -y openssl
    O que mais pode.
      echo "Seu gerenciador de pacotes não é suportado. Por favor, instale o openssl manualmente."
      saída 1
    fi
  O que mais pode.
    "Por favor, instale openssl, shasum ou sha256sum e tente novamente."
    saída 1
  fi
fi


“Durante este estágio inicial da Betanet, a equipe de Shardeum coletará algum desempenho e depuração de informações do seu nó para ajudar a melhorar as versões futuras do software.
Isso é apenas temporário e será descontinuado à medida que nos aproximamos da mainnet.
Obrigado por correr um nó e ajudar a fazer Shardeum melhor.

Ao executar este instalador, você concorda em permitir que a equipe Shardeum colete esses dados. (Y/n)?: "AVISO ACORDO
AVISO_CONEU PRÉDOS (eco "$WARNING_ACOR" | tr '[:upper:]'''
AVISO_ACORDO ?$????????

se [ $ARNING_ACORDO! ? "y" ];
Em seguida,
  echo "Concesso de recolha de dados diagnóstico não aceite. Saída do instalador."
  Saída
fi

read -p "Que diretório base deve o uso do nó (padrão ?/.shardeum): " entrada

Definir valor padrão se a entrada estiver vazia
inputTM$?Binput:-/.shardeum

- Verifica se a entrada começa com "/" ou "-/", se não, adicionar "-/"
Se [[ ! $input ? ?(/|/) ]]; em seguida,
  input "/$input"
fi

Reprompt se não caracteres alfanuméricos, tilde, barra para a frente, sublinhado, ponto longo, hífen, ou contém espaços
enquanto [[ ! $input ?[[:alnum:]_./-]+$ | $input ? ? ? ? ? ? ? ? ]; do
  read -p "Erro: O nome do diretório contém caracteres ou espaços inválidos.
Os caracteres permitidos são caracteres alfanuméricos, tilde, barra para a frente, sublinhado, período e hífen.
Por favor, insira um diretório base válido (padrão ?/.shardeum): " entrada

  - Verifica se a entrada começa com "/" ou "-/", se não, adicionar "-/"
  Se [[ ! $input ? ?(/|/) ]]; em seguida,
    input "/$input"
  fi
feito

- Remover espaços da entrada
inputTM$?B.input// /?

Echo o diretório final usado
eco "O diretório base está definido como: $input"

Substituir o tilde líder (?) pelo caminho real do diretório inicial
NODEHOME?"$?Binput//$HOME? " ? suporte ? em caminho

Verifique todas as coisas que serão necessárias para que este script tenha sucesso como acesso ao docker e docker-compose
Se qualquer verificação falhar sair com uma mensagem sobre o que o usuário precisa fazer para corrigir o problema
comando -v git ?/dev/null 2'&1 || ? ? echo &2 "'git' é necessário, mas não instalado."; exit 1;
comando -v docker ?/dev/null 2'&1 || ? ? ?&2 "'docker' é necessário, mas não está instalado. Veja https://gitlab.com/shardeum/validator/dashboard/-tree/dashboard-gui-nextjs-how-to para detalhes."; saída 1;
se comando -v docker-compose &/dev/null; então
  echo "docker-compose é instalado nesta máquina"
elif docker --help | grep -q "compor"; então
  Echo "docker compose subcommand é instalado nesta máquina"
O que mais pode.
  echo "docker-compose ou docker compose não está instalado nesta máquina"
  saída 1
fi

exportação DOCKER_DEFAULT_PLATFORM-linux/amd64

docker-safe() ?
  se ! comando -v docker & / dev/null; então
    echo "docker não está instalado nesta máquina"
    saída 1
  fi

  se ! docker $; então
    echo "Trying again com sudo..." ?&2 (em inglês)
    Sudo docker $
  fi
) ) Em (em,

docker-compose-safe() ?
  se comando -v docker-compose &/dev/null; então
    cmd'docker-compose" (em inglês)
  elif docker --help | grep -q "compor"; então
    cmd'docker compose"
  O que mais pode.
    echo "docker-compose ou docker compose não está instalado nesta máquina"
    saída 1
  fi

  Se for! $cmd $ $ ?; em seguida,
    echo "Trying again com sudo..."
    Sudo $cmd $ $ ?
  fi
) ) Em (em,

get_ip() (em inglês) ?
  local ip
  se comando -v ip -/dev/null; em seguida,
    ip'$(ip addr show $(ip route | awk '/default/ ?print $5'') | awk '/inet/ ?print $2'' | cut -d/ -f1 | head -n1)
  comando elif -v netstat -/dev/null; em seguida,
    Obter a interface de rota padrão
    interfaceTM$(netstat -rn | awk '/default/'impressão de US$ 4'' | head -n1)
    Obter o endereço IP para a interface padrão
    ip'$(ifconfig "$interface" | awk '/inet /'print $2'')
  O que mais pode.
    echo "Erro: nem 'ip' nem comando 'ifconfig'' encontrado. Envie um bug para o seu sistema operacional."
    retorno 1
  fi
  echo $ipTradução
) ) Em (em,

get_external_ip() ?
  external_ip?' (')
  external_ipTM(curl -s https://api.ipify.org)
  se [[ -z "$external_ip"]]; então
    external_ip?$(curl -s http://checkip.dyndns.org | grep -o "??([0-9] ?1,3?.)
  fi
  se [[ -z "$external_ip"]]; então
    external_ip?$(curl -s http://ipecho.net/plain)
  fi
  se [[ -z "$external_ip"]]; então
    external_ipTM(curl -comira https://icanhazip.com/)
  fi
    se [[ -z "$external_ip"]]; então
    external_ipTM$(curl --header "Host: icanhazip.com" -es 104.18.114.97)
  fi
  se [[ -z "$external_ip"]]; então
    external_ip?$(get_ip)
    se [ $? -eq 0 ]; então
      echo "O endereço IP é: $IP"
    O que mais pode.
      external_ip?"localhost" (em inglês)
    fi
  fi
  echo $external_ip
) ) Em (em,

hash_password()
  input local: "$1"
  local hashed_password

  - Tente usar o openssl
  se comando -v openssl - /dev/null; então
    hashed_passwordTM$(echo -n "$input" | openssl dgst -sha256 -r | awk '?imprint $1?')
    echo "$hashed_password" (em inglês)
    retorno 0
  fi

  - Tente usar shasum
  se comando -v shasum ? /dev/null; então
    hashed_passwordTM$(echo -n "$input" | shasum -a 256 | awk '?imprimir $1')
    echo "$hashed_password" (em inglês)
    retorno 0
  fi

  Tente usar sha256sum
  se comando -v sha256sum ? /dev/null; então
    hashed_passwordTM$(echo -n "$input" | sha256sum | awk '?print $1')
    echo "$hashed_password" (em inglês)
    retorno 0
  fi

  retorno 1
) ) Em (em,

se [[ $(docker-safe info 2'&1) ???Não pode ligar ao Docker daemon" ? ]; então
    echo "Docker daemon não está funcionando"
    saída 1
O que mais pode.
    echo "Docker daemon está correndo"
fi

CURRENT_DIRECTORY$(pwd)

VALÁVEIS DEFAULT PARA INPUTAS DE USUÁRIO
DASHPORT_DEFAULT ? 8080
EXTERNALIP_DEFAULT
INTERNALIP_DEFAULT-auto
SHMEXT_DEFAULT ?9001
SHMINT_DEFAULT ? 10001
PREVIOUS_PASSWORD-none

Verificar se o recipiente existe
IMAGEM ?registry.gitlab.com/shardeum/server: mais recente"
CONTAINER_IDTM$(docker-safe ps -qf "ancestor-local-dashboard")
se [ ! -z "$-CONTAINER_ID" ]; então
  echo "CONTAINER_ID: $-CONTAINER_ID"
  echo "Recipiente existente encontrado. Leitura de configurações do contêiner."

  Atribuir saída de read_container_settings para variável
  Se for! ENV_VARSTM$(docker inspect --format'''range .Config.Env?println .?end?" "$CONTAINER_ID");
    ENV_VARSTM$(sudo docker inspecion --formatt'""range .Config.Env.println .??enc.??" "$CONTAINER_ID")
  fi

  se ! docker-safe cp "$-CONTAINER_ID:/home/node/app/cli/build/secrets.json" ./;
    echo "Container não tem segredos.json"
  O que mais pode.
    echo "Reutilizando segredos.json do recipiente"
  fi

  CONTA ESCRA IFIDATOR SE VALIDATOR IRÁ PARA A JUNDA
  set +e
  status$(docker-safe exec "$-CONTAINER_ID" operator-cli status 2'/dev/null)
  Check-mail $?
  set -e

  se [ $check -eq 0 ]; então
    O comando foi executado com sucesso
    status$(awk '/state:/ ?print $ 2'' ? $status)
    se [ "$status" ? "active" ] ||| [ "$status" ? "sinício" ]; então
      “Seu nó é $status e a atualização fará com que o nó deixe a rede inesperadamente e perca o valor da participação.
      Você realmente quer atualizar agora (y/N)?" REALMENTE AGRADECIMENTO
      REALLYUPGRADE$(echo "$REALLYUPGRADE" | tr '[:upper:]' '[:lower:]')
      REALLYUPGRADE$-REALLYUPGRADE:-n

      se [ "$REALLYUPDE" ? "n" ]; então
        saída 1
      fi
    O que mais pode.
      echo "O processo do valedor não está online"
    fi
  O que mais pode.
    "O instalador não foi capaz de determinar se o nó existente está ativo.
    Um nó ativo inesperadamente deixando a rede perderá o valor da sua aposta.
    Você realmente quer atualizar agora (y/N)?" REALMENTE AGRADECIMENTO
    REALLYUPGRADE$(echo "$REALLYUPGRADE" | tr '[:upper:]' '[:lower:]')
    REALLYUPGRADE$-REALLYUPGRADE:-n

    se [ "$REALLYUPDE" ? "n" ]; então
      saída 1
    fi
  fi

  docker-safe stop "$-CONTAINER_ID"
  docker-safe rm "$-CONTAINER_ID"

  UPDATE DEFAULT VALUES COM VALORES SAVED
  DASHPORT_DEFAULTTM$(echo $ENV_VARS | grep -oP 'DASHPORT-K[ ? ?]+') || DASHPORT_DEFAULT
  EXTERTERNALIP_DEFAULT$(echo $ENV_VARS | grep -oP 'EXT_IP-K[-+') || EXTERNALIP_DEFAULTTMauto
  INTERNALIP_DEFAULT$(echo $ENV_VARS | grep -oP 'INT_IP'[ ?]+') || INTERNALIP_DEFAULT
  SHMEXT_DEFAULT$(echo $ENV_VARS | grep -oP 'SHMEXT-K[-+') || SHMEXT_DEFAULT
  SHMINT_DEFAULT$(echo $ENV_VARS | grep -oP 'SHMINT?K[? ?]+') || SHMINT_DEFAULT
  PREVIOUS_PASSWORD$(echo $ENV_VARS | grep -oP 'DASHPASS?K[ ]+') ||ANTERIOS_PASWORD
elif [ -f NODEHOME/.ptv ];
  eco "Existendo arquivo NODEHOME/.env encontrado. Leitura de configurações do arquivo."

  - Leia o arquivo NODEHOME/.env em uma variável. Use o diretório do instalador padrão se existir.
  ENV_VARS ?$(ga NODEHOME/.ptv)

  UPDATE DEFAULT VALUES COM VALORES SAVED
  DASHPORT_DEFAULTTM$(echo $ENV_VARS | grep -oP 'DASHPORT-K[ ? ?]+') || DASHPORT_DEFAULT
  EXTERTERNALIP_DEFAULT$(echo $ENV_VARS | grep -oP 'EXT_IP-K[-+') || EXTERNALIP_DEFAULTTMauto
  INTERNALIP_DEFAULT$(echo $ENV_VARS | grep -oP 'INT_IP'[ ?]+') || INTERNALIP_DEFAULT
  SHMEXT_DEFAULT$(echo $ENV_VARS | grep -oP 'SHMEXT-K[-+') || SHMEXT_DEFAULT
  SHMINT_DEFAULT$(echo $ENV_VARS | grep -oP 'SHMINT?K[? ?]+') || SHMINT_DEFAULT
  PREVIOUS_PASSWORD$(echo $ENV_VARS | grep -oP 'DASHPASS?K[ ]+') ||ANTERIOS_PASWORD
fi

gato ? EOF

) ) Em (em,
? 0. GET INFO FROM USUÁRIO ?
) ) Em (em,

EOF (em inglês)

"Você quer executar o painel baseado na web? (Y/n): " RUNDASHBOARD
RUNDASHBOARD$$(echo "$RUNDASHBOARD" | tr '[:upper:]' '[:lower:]')
RUNDASHBOARDTM$?RUNDASHBOARD:-y

se [ "$PREVIOUS_PASSWORD" ! "none" ]; então
  "Você quer alterar a senha do Painel? (y/N): " MUDÁS COMPRAS
  CHANGEPASSWORD$(eeco "$CHANGEPASSWORD" | tr '[:upper:]' '[:lower:]')
  CHANGEPASSWORD$-CHANGEPASSWORD:-n
O que mais pode.
  CHANGEPASSWORD"y
fi

se [ "$CHANGEPASSWORD" - "y" ]; então
  CHARCOUNT desativado
  echo -n "Defina a senha para acessar o Painel: "
  CHARCOUNT ? 0
  enquanto IFS ler -p "$PROMPT" -r -s -n 1 CHAR
  Faça o que fazer.
    - Digite - aceite a senha
    se [[ $CHAR ? $' 0' ] ; em seguida,
      se [ $CHARCOUNT -gt 0 ] ; em seguida, certifique-se de que o comprimento do caractere de senha é maior que 0.
        quebrar
      O que mais pode.
        Echo
        echo -n "Invalida de senha inválida. Digite uma senha com comprimento de caractere maior que 0:"
        Continue
      fi
    fi
    ? Backspace (em inglês)
    se [[ $CHAR ? $' $'177' ] ] ; em seguida ; em seguida
      se [ $CHARCOUNT -gt 0 ] ; então
        CHARCOUNT$((CHARCOUNT-1))
        PROMPTTM$'b ?b'
        DASHPASSTM"$.DASHPASS%?
      O que mais pode.
        PROMPT'' (em inglês)
      fi
    O que mais pode.
      CHARCOUNTTM$((CHARCOUNIZAçãO+1))
      PROMPT''
      DASHPASS+-"$CHAR"
    fi
  feito

  Hash a senha usando o mecanismo fallback
  DASHPASSTM$(hash_password "$DASHPASS")
O que mais pode.
  DASHPASSTM$PREVIOUS_PASSWORD
  Se for! [[ $DASHPASS ? ? ?[0-9a-f]?64?$ ]]]; então
    DASHPASSTM$(hash_password "$DASHPASS")
  fi
fi

se [ -z "$DASHPASS" ]; então
  echo -e "?nFailed para hash a senha. Por favor, certifique-se de que você tem openssl"
  saída 1
fi

eco ? Nova linha após as entradas.
- eco "Senha salva como:" $DASHPASS ?DEBUS: TEST PASSWORD WAS RECORDED DEPOIS DE DESCONTO.

enquanto :; faça
  leia-se "Entrar na porta (1025-65536) para acessar o Painel baseado na web (padrão $DASHPORT_DEFAULT): " DASHPORT
  DASHPORTTM$?DASHPORT:-$DASHPORT_DEFAULT
  [[$DASHPORT ? ? ?[0-9]+$ ]] || ? ? echo "Enter a porta válida"; continue;
  se ((DASHPORT ? 1025 && DASHPORT ? 65536));
    DASHPORTTM$?DASHPORT:-$DASHPORT_DEFAULT
    quebrar
  O que mais pode.
    eco "Port out of range, try again"
  fi
feito

enquanto :; faça
  read -p "Se você deseja definir um IP externo explícito, digite um endereço IPv4 (padrão $EXTERNALIP_DEFAULT): "External
  EXTERNALIPTM$?EXTERNALIP:-$EXTERNALIP_DEFAULT

  se [ "$EXTERNALIP" - "auto" ]; então
    quebrar
  fi

  Use regex para verificar se a entrada é um endereço IPv4 válido
  se [[ $EXTERNALIP ? ?(0-9]?1,3?.)? 3? [0-9] ? 1,3?$ ]]; em seguida;
    Verifique se cada número no endereço IP está entre 0-255
    valid_ip-true (tradução)
    IFS'.'. read -ra ip_nums ? "$EXTERNALIP"
    para o número em "$?ip_nums[?]"
    Faça o que fazer.
        se (( num ? 0 || num ? 255 )); em seguida,
            valid_ip?false
        fi
    feito

    se [ $valid_ip ? true ]; então
      quebrar
    O que mais pode.
      echo "Disputo IPv4 inválido. Por favor, tente de novo."
    fi
  O que mais pode.
    echo "Disputo IPv4 inválido. Por favor, tente de novo."
  fi
feito

enquanto :; faça
  read -p "Se você deseja definir um IP interno explícito, digite um endereço IPv4 (padrão $INTERNALIP_DEFAULT): " INTERNALIP
  INTERNALIPTM$?INERNALIP:-$INTERNALIP_DEFAULT

  se [ "$INTERNALIP" - "auto" ]; então
    quebrar
  fi

  Use regex para verificar se a entrada é um endereço IPv4 válido
  se [[ $INTERNALIP ? ?((0-9] ?1,3?.)? 3? [0-99] ? 1,3 $ ]]; em seguida,
    Verifique se cada número no endereço IP está entre 0-255
    valid_ip-true (tradução)
    IFS'.'. read -ra ip_nums ? "$INTERNALIP"
    para o número em "$?ip_nums[?]"
    Faça o que fazer.
        se (( num ? 0 || num ? 255 )); em seguida,
            valid_ip?false
        fi
    feito

    se [ $valid_ip ? true ]; então
      quebrar
    O que mais pode.
      echo "Disputo IPv4 inválido. Por favor, tente de novo."
    fi
  O que mais pode.
    echo "Disputo IPv4 inválido. Por favor, tente de novo."
  fi
feito

enquanto :; faça
  Para executar um validador na rede Sphinx, você precisará abrir duas portas no firewall.
  leia -p "Isso permite a comunicação p2p entre nós. Digite a primeira porta (1025-65536) para comunicação p2p (padrão $SHMEXT_DEFAULT): " SHMEXT
  SHMEXTTM$-SHMEXT:-$SHMEXT_DEFAULT
  [[$SHMEXT ? ?[0-9]+$ ]] || ? eco "Inserir uma porta válida"; continuar;
  se ((SHMEXT ? 1025 & & SHMEXT ? 65536)); em seguida,
    SHMEXTTM$-SHMEXT:- 9001
  O que mais pode.
    eco "Port out of range, try again"
  fi
  Leia -p "Entre na segunda porta (1025-65536) para comunicação p2p (padrão $SHMINT_DEFAULT): " SHMINT
  SHMINTTM$-SHMINT:-$SHMINT_DEFAULT
  [[$SHMINT ? ?[0-9]+$ ]] || ? echo "Enter a porta válida"; continue;
  se ((SHMINT ? 1025 & & SHMINT ? 65536));
    SHMINTTM$-SHMINT:-10001
    quebrar
  O que mais pode.
    eco "Port out of range, try again"
  fi
feito

?APPSEEDLIST?"archr-sphinx.shardeum.org" (em inglês)
?APPMONITOR?"monitor-sphinx.shardeum.org" (em inglês)
APPMONITOR"104.200.21.5"

gato ?EOF

) ) Em (em,
? ? (em inglês). Puxe o Projeto de Composição ?
) ) Em (em,

EOF (em inglês)

se [ -d "$NODEHOME" ]; então
  se [ "$NODEHOME" ! - "$(pwd)" ]; então
    echo "Remover diretório existente $NODEHOME..."
    rm -rf "$NODEHOME"
  O que mais pode.
    echo "Não é possível excluir o diretório de trabalho atual. Por favor, mova para outro diretório e tente novamente."
  fi
fi

git clone https://gitlab.com/shardeum/validator/dashboard.git $?NODEHOME? || ? eco "Error: Permission negou. Roteiro de saída."; saída 1; ?
cd $?NODEHOME
chmod a+x ./.sh

gato ?EOF

) ) Em (em,
? 2. Criar e Definir o Ficheiro .env
) ) Em (em,

EOF (em inglês)

SERVERIPTM$(get_external_ip))
LOCALLANIPTM$(get_ip) (em inglês)
cd $?NODEHOMETM &&
Toque ./.ptv
gato ./.ptv ?EOL
EXT_IPTM$?EXTERNALIP
INT_IPTM$?INTERNALIP
EXISTING_ARCHIVERS["ip":"45.56.68.62","porto":4000,"público":"840e7b59a59a95d3c5f504f64f62ab9fab9fab10107d39100114141098EX2e3cde63,"ip": "pt.173.255.247.88"aad41c306f63ee972655121a37c5e4f52b00a542","ip":"170.187.154.43", "porto":4000,"Chapo-público":"7af699d711074eb96a8d1103e32b2b52b58916ebb0c6a89a9e87e87
APP_MONITORTM$?APPMONITOR
DASHPASSTM$.DASHPASSTM
DASHPORTTM$?DASHPORTTM
SERVERIPTM$?SERVERIP?
LOCALLANIPTM$-LOCALLANIPTM
SHMEXTTM$?SHMEXTTM
SHMINTTM$?SHMINTTM
EOL (em inglês)

gato ?EOF

) ) Em (em,
? 3. Clearing Imagens antigas ?
) ) Em (em,

EOF (em inglês)

./cleanup.sh (em inglês)

gato ?EOF

) ) Em (em,
? 4. Imagem base de construção ?
) ) Em (em,

EOF (em inglês)

cd $?NODEHOMETM &&
docker-safe build --no-cache -t local-dashboard -f Dockerfile --build-arg RUNDASHBOARDTM$RUNDASHBOARD .

gato ?EOF

) ) Em (em,
? 5. (em inglês). Iniciar o Compose Project ?
) ) Em (em,

EOF (em inglês)

cd $?NODEHOME
se [[ "$(uname)" ? "Darwin"]]; então
  sed "s/- '8080:8080'/- '$DASHPORT:$DASHPORT'/" docker-compose.tmpl ? docker-compose.yml
  sed -i '' "s/- '9001-9010:9001-9010'/- '$SHMEXT:$SHMEXT'/" docker-compose.yml
  sed -i '' "s/- '10001-10010:10001-10010'/- '$SHMINT:$SHMINT'/" docker-compose.yml
O que mais pode.
  sed "s/- '8080:8080'/- '$DASHPORT:$DASHPORT'/" docker-compose.tmpl ? docker-compose.yml
  sed -i "s/- '9001-9010:9001-9010'/- '$SHMEXT:$SHMEXT'/" docker-compose.yml
  sed -i "s/- '10001-10010:10001-10010'/- '$SHMINT:$SHMINT'/" docker-compose.yml
fi
./docker-up.sh (em inglês)

echo "Alargar imagem. Isso pode levar um tempo..."
(registres seguros para o dobr -f shardeum-dashboard &) | grep -q 'done'

Verifique se secrets.json existe e copiá-lo dentro do recipiente
cd $ ?CURRENT_DIRECTORY
se [ -f secrets.json]; então
  echo "Reutilizando o velho nó"
  CONTAINER_IDTM$(docker-safe ps -qf "ancestor-local-dashboard")
  echo "O novo id do recipiente é: $CONTAINER_ID"
  docker-safe cp ./secrets.json "$-CONTAINER_ID:/home/node/cli/compilação/secrets.json"
  rm -f secrets.json
fi

?Não recuo
se [ $RUNDASHBOARD ? "y" ]
Em seguida,
gato ?EOF
  Para usar o Web Dashboard:
    1. Observe o endereço IP que você usou para conectar ao nó. Isso pode ser um IP externo, IP LAN ou localhost.
    2. Abra um navegador da Web e navegue até o painel da Web em https://?Node IP endereço:$DASHPORT
    3. Vá para a guia Configurações e conecte uma carteira.
    4. Vá para a guia Manutenção e clique no botão Iniciar o nó.

  Se este validador está na nuvem e você precisa chegar ao painel através da internet,
  Por favor, defina uma senha forte e use o IP externo em vez de localhost.
EOF (em inglês)
fi

gato ?EOF

Para usar a interface da linha de comando:
	1. Navegue até o diretório home do Shardeum ($NODEHOME).
	2. Digite o recipiente do validador com ./shell.sh.
	3. Executar "operator-cli --help" para comandos

EOF (em inglês)
