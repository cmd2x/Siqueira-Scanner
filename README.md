# Siqueira Scanner

Scanner de triagem para Windows, com interface gráfica nativa. Reúne verificações de processos, drivers, inicialização, rastros locais e eventos do Sysmon em uma única execução.



## O que ele verifica

A varredura automática executa 27 módulos:

- **Sistema e configuração:** versão, build, arquitetura, firmware UEFI/BIOS, uptime, Secure Boot e integridade de memória.
- **Rastros locais:** histórico de dispositivos USB, atalhos recentes, itens relevantes da Lixeira e configurações de políticas do Windows.
- **ADB:** processos `adb.exe` e `hd-adb.exe` em execução e comandos associados a operações como `shell`, `push`, `pull`, `connect` e `tcpip`.
- **Serviços e processos:** estado de serviços do Windows e Sysmon; processos e módulos carregados; indicadores conhecidos, módulos duplicados em emuladores e módulos carregados de diretórios graváveis pelo usuário.
- **Artefatos conhecidos:** nomes e strings associados a loaders, overlays, drivers e outros indicadores estáticos. A busca em arquivos tem limites de diretórios, profundidade e tamanho.
- **Janelas:** títulos que correspondem a termos monitorados e janelas com afinidade de captura configurada.
- **Visibilidade de processos (DKOM):** compara cinco vezes as listas de processos obtidas por EnumProcesses, Toolhelp, NtQuerySystemInformation e enumeração de janelas. Divergências podem ser transitórias ou inconclusivas.
- **Memória de processos:** procura detect`s nas listas de módulos, mapas de memória e páginas residentes dos processos acessíveis.
- **Rastros de execução:** entradas BAM desde o boot, histórico de execução no Registro, arquivos Prefetch removidos identificáveis pelo USN Journal e disponibilidade do USN Journal em volumes fixos NTFS.
- **Assinaturas:** verifica Authenticode de drivers de kernel registrados e faz triagem de arquivos sem assinatura ou com assinatura inválida em unidades fixas.
- **Boot e EFI:** consulta configurações BCD, Secure Boot e eventos de Kernel-Boot/Code Integrity; procura hashes e indicadores conhecidos em caminhos EFI montados e drivers.
- **Windows Defender:** consulta estado do serviço, antivírus, proteção em tempo real, Tamper Protection e versão das assinaturas, quando disponíveis.
- **Logs do Windows:** procura eventos relacionados à limpeza dos logs e registra eventos de erro de aplicação encontrados.
- **Instalação de drivers:** revisa eventos de sistema 7045 e destaca caminhos ou nomes que correspondem a termos monitorados.
- **Sysmon:** verifica serviço e configuração e analisa eventos:
  - ID 1: padrões como PowerShell codificado ou oculto, download e execução, comandos de limpeza de logs e registro remoto de scriptlets.
  - ID 5: processos encerrados recentemente.
  - ID 7: módulos conhecidos e imagens sem assinatura em caminhos graváveis pelo usuário.
  - IDs 8 e 10: indicadores de criação de thread remota e acesso ao processo `HD-Player.exe`.
  - ID 16: alterações de configuração do Sysmon.
  - ID 22: consultas DNS que correspondem a domínios ou palavras-chave monitorados.
- **Timeline Sysmon:** aplica quatro regras a eventos exportados: processo encerrado sem evento de criação correspondente, acesso ao `HD-Player.exe`, criação de thread remota envolvendo esse processo e anomalia no processo pai de imagens do Windows.

## Interface

- **Painel:** progresso, módulos e resultados durante a varredura.
- **Resultados:** achados classificados como Detectado, Aviso, OK ou Info; permite filtrar por texto e status e copiar itens.
- **Pesquisa Sysmon:** consulta por IDs e texto, com período e limite de resultados. Um campo de IDs vazio pesquisa todos os eventos disponíveis. Os resultados podem ser exportados como CSV.
- **Console:** exibe o log completo e permite copiar ou salvar a saída em `.txt`.
- **CMD:** console interativo do Windows disponível na interface.
- **Informações:** exibe HWID, usuário, computador, sistema operacional, privilégios, horário do último boot, estado do Sysmon e Secure Boot.

## Arquivos gerados

A timeline Sysmon cria, ao lado do executável, a pasta `sysmon_dump` com:

- `sysmon.evtx`: exportação local do canal operacional do Sysmon.
- `timeline.csv`: alertas produzidos pelas quatro regras da timeline.

A pesquisa Sysmon também pode exportar seus resultados para um CSV escolhido pelo usuário. O log do console pode ser salvo em `.txt`. A exportação da timeline substitui os arquivos de mesmo nome existentes nessa pasta.

## Limitações e privacidade

- Os resultados dependem dos logs, permissões, configurações e indicadores disponíveis na máquina. Leituras bloqueadas ou fontes ausentes podem deixar verificações incompletas.
- Arquivos sem assinatura, nomes suspeitos, divergências de processos e consultas DNS monitoradas podem ser legítimos. Revise cada achado antes de tomar qualquer medida.
- A análise em modo usuário não comprova nem descarta todas as formas de manipulação do kernel. A busca EFI cobre caminhos montados; partições ocultas ou não montadas não são inspecionadas.
- A varredura não remove, bloqueia ou coloca arquivos em quarentena. A timeline, porém, exporta logs e grava arquivos locais.
- A rotina de varredura não implementa envio de resultados para um servidor. A interface inclui um CMD interativo que executa comandos com os privilégios do programa; comandos digitados ali podem alterar o sistema ou acessar a rede.
- Os relatórios podem conter HWID, nome de usuário e computador, caminhos, processos e dados de eventos. Revise e remova informações pessoais antes de publicar arquivos no GitHub.

## Requisitos

- Windows 10/11 x64.
- Sysmon instalado e configurado para as verificações baseadas em eventos. Sem Sysmon, os demais módulos ainda podem ser executados, mas as consultas correspondentes ficam indisponíveis.
- Execute como administrador para ampliar o acesso a serviços, eventos e outras evidências. A exportação da timeline Sysmon exige privilégios de administrador.
