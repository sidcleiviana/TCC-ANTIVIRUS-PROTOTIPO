# TCC-ANTIVIRUS-PROTOTIPO
PRIMEIRO DE TUDO, RODEI O arquivo python utils/generate_hashes.py, nele o Python executa este bloco do código: 
if __name__ == "__main__":
    samples_path = "samples"
    generate_hashes(samples_path)
Ou seja: primeiro define que a pasta analisada será a pasta samples e depois chama a função: generate_hashes("samples")

E o que que essa função faz? Vai percorrer todos os arquivos dentro da pasta samples.
Nessa linha “for root, dirs, files in os.walk(directory):” O os.walk() significa: percorrer todos os arquivos e subpastas de um diretório.
Para cada arquivo encontrado Ele executa: file_hash = calculate_sha256(file_path)
Aqui ele chama a função que calcula o hash usando SHA-256. Aí essa função está em outro arquivo do projeto.
Entao simplificando,
rodamos o 
 generate_hashes.py
        ↓
ele abre pasta samples
        ↓
pega cada arquivo
        ↓
envia para hashing.py
        ↓
hashing.py calcula SHA256
        ↓
retorna hash
        ↓
generate_hashes imprime resultado


Entao na pratica ficou assim, rodei o: python -m utils.generate_hashes
E ela gerou: 

E então criei a base de assinaturas. Observe a hash do fake_malware.exe… É ela que temos que tomar cuidado, no arquivo scanner/signatures.py tem o codigo assim:
# Lista de hashes conhecidos como maliciosos
MALWARE_HASHES = {
    " 2f13a19fe1f1423034ce6cab99d69c3ee1b55d28fbabe7bc719cb12719562272": "Teste falso de Malware"
}
Somente fake_malware.exe entra na base. Os outros serão detectados por heurística.
Ou seja, esse ataque já não afetará mais a máquina que nosso anti vírus está protegendo.

Como colocamos essa hash na nossa base de assinatura. E AI quando rodarmos o antivírus, que é o arquivo main.py; 
from scanner.scanner import scan_directory
scan_directory("samples")

A saída esperada é que reconheça esse arquivo samples\fake_malware.exe como malicioso.
E foi exatamente isso que aconteceu:


Aí vamos para a parte de detecção heurística
A detecção heurística é uma técnica utilizada por antivírus para identificar possíveis ameaças mesmo quando elas não possuem uma assinatura conhecida. Diferentemente da detecção por assinatura, que depende da comparação de hashes previamente cadastrados, a heurística analisa características e comportamentos potencialmente suspeitos presentes em arquivos.
Entre esses comportamentos podem estar scripts que executam comandos perigosos, tentativas de manipulação de arquivos sensíveis ou padrões frequentemente associados a malware.
Embora a análise heurística não garanta a identificação exata de um malware específico, ela permite classificar arquivos como potencialmente suspeitos, permitindo ações preventivas como alerta ao usuário ou isolamento em quarentena.
e o que a heurística vai detectar
condição
classificação
arquivo .bat
script suspeito
arquivo contendo palavra encrypt
possível ransomware
arquivo contendo delete
comportamento perigoso

Isso vai simular comportamento de malware.

No arquivo heuristics/heuristics.py nós temos uma função completa.
def analyze_file(file_path):
    try:
        with open(file_path, "r", errors="ignore") as f:
            content = f.read().lower()


        if file_path.endswith(".bat"):
            return "Script potencialmente perigoso"


        if "encrypt" in content:
            return "Comportamento semelhante a ransomware"


        if "del " in content or "delete" in content:
            return "Comando de exclusão detectado"


        return None


    except:
        return None

Essa função realiza uma análise heurística simples, abrindo o arquivo e verificando padrões suspeitos em seu conteúdo. Por exemplo, a condição if file_path.endswith(".bat") identifica scripts potencialmente perigosos, enquanto verificações como if "encrypt" in content e if "del " in content buscam indícios de comportamentos maliciosos, como criptografia de arquivos ou comandos de exclusão de dados. Caso algum desses padrões seja encontrado, a função retorna uma descrição do comportamento suspeito.


Aí o fluxo é o seguinte, vou rodar o:
main.py
   ↓
interface/cli.py → mostra menu
   ↓
scanner/scanner.py → analisa arquivo ou diretório
   ↓
utils/hashing.py → calcula SHA256
   ↓
scanner/signatures.py → compara assinatura
   ↓
heuristics/heuristics.py → analisa comportamento suspeito

E na prática é assim: na CLI O ANTI VIRUS VAI PERGUNTAR O QUE O VOCÊ QUER FAZER:


Escolhi escanear o diretorio, e escolhi o diretoria onde fica os arquivos que criamos. 

Vemos então 2 arquivos OK, 2 SUSPEITOS e 1 em ALERTA. 


Pra finalizar o projeto, uma parte muito importante que é a quarentena é um mecanismo utilizado por softwares antivírus para isolar arquivos identificados como maliciosos ou suspeitos.
Em vez de excluir imediatamente o arquivo, o sistema o move para um diretório seguro onde não pode ser executado. Dessa forma, o risco de infecção é reduzido, enquanto o arquivo permanece disponível para análise posterior ou restauração caso necessário.
No protótipo desenvolvido neste trabalho, arquivos detectados como malware são automaticamente movidos para um diretório de quarentena, onde são armazenados e registrados para fins de auditoria.
Assim fica a estrutura da pasta

Inicialmente, a instrução os.makedirs(QUARANTINE_DIR, exist_ok=True) garante que o diretório de quarentena exista, criando-o caso necessário. Em seguida, o nome do arquivo é obtido por meio de os.path.basename(file_path) e o destino final é definido com os.path.join(QUARANTINE_DIR, file_name). Por fim, o arquivo é transferido para essa pasta utilizando shutil.move(file_path, destination), impedindo que ele permaneça em seu local original e reduzindo o risco de execução acidental.
Simplificando: ele vai criar a pasta de quarentena se não existir, pega o nome do arquivo, e move o arquivo pasta quarantine/quarantine_files

E AQUI ESTÁ O TESTE FINAL COMPLETO




ESSE PROJETO ENTÃO REPRESENTA

mecanismo
implementado
detecção por assinatura
✔
análise heurística
✔
quarentena
✔


Arquivo → Scanner
       ↓
Cálculo SHA256
       ↓
Comparação com assinaturas
       ↓
Se não detectar →
Análise heurística
       ↓
Se suspeito →
Mover para quarentena

O desenvolvimento deste projeto permitiu demonstrar, de forma prática, o funcionamento de alguns dos principais mecanismos utilizados em soluções de antivírus. Ao longo do trabalho, foi possível implementar um protótipo funcional capaz de realizar a varredura de arquivos, identificar ameaças por meio de assinaturas baseadas em hash SHA-256, aplicar uma análise heurística simplificada e isolar arquivos suspeitos através de um mecanismo de quarentena.
Embora o sistema desenvolvido tenha caráter acadêmico e simplificado quando comparado a soluções comerciais, ele evidencia conceitos fundamentais da área de segurança da informação, permitindo compreender como diferentes técnicas podem ser combinadas para identificar e mitigar ameaças digitais.
Além disso, a arquitetura modular adotada no projeto possibilita futuras expansões, como a inclusão de monitoramento em tempo real, atualização automática da base de assinaturas ou integração com técnicas mais avançadas de análise comportamental e aprendizado de máquina.
Dessa forma, o trabalho contribui para a compreensão dos princípios básicos envolvidos no desenvolvimento de sistemas de proteção contra malware, demonstrando que, mesmo em uma implementação simplificada, é possível reproduzir elementos essenciais presentes em softwares antivírus modernos.

