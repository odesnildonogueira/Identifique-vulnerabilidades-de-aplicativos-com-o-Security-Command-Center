# Identifique-vulnerabilidades-de-aplicativos-com-o-Security-Command-Center
SOLUÇÃO DO PROBLEMA Google Cloud CyberSecurity Certificate - Curso 3 - Módulo 2 - Identificar vulnerabilidades e técnicas de correção
### Na barra de título do Console do Google Cloud, clique em Ativar Cloud Shell (ícone Ativar Cloud Shell). Se solicitado, clique em Continuar.

#### EXPORT ZONE

```bash
export ZONE=
```


####

```bash
export REGION="${ZONE%-*}"

gcloud services enable websecurityscanner.googleapis.com

gcloud compute addresses create xss-test-ip-address --region=$REGION

gcloud compute addresses describe xss-test-ip-address \
--region=$REGION --format="value(address)"

gcloud compute instances create xss-test-vm-instance \
--address=xss-test-ip-address --no-service-account \
--no-scopes --machine-type=e2-micro --zone=$ZONE \
--metadata=startup-script='apt-get update; apt-get install -y python3-flask'

gcloud compute firewall-rules create enable-wss-scan \
--direction=INGRESS --priority=1000 \
--network=default --action=ALLOW \
--rules=tcp:8080 --source-ranges=0.0.0.0/0

sleep 20

EXTERNAL_IP=$(gcloud compute instances describe xss-test-vm-instance --zone=$ZONE --format="get(networkInterfaces[0].accessConfigs[0].natIP)")

gcloud alpha web-security-scanner scan-configs create --display-name=quicklab --starting-urls=http://$EXTERNAL_IP:8080


SCAN_CONFIG=$(gcloud alpha web-security-scanner scan-configs list --project=$DEVSHELL_PROJECT_ID --format="value(name)")

gcloud alpha web-security-scanner scan-runs start $SCAN_CONFIG
```





### Vá para a instância da VM e clique em ssh e execute o código 

```bash
gsutil cp gs://cloud-training/GCPSEC-ScannerAppEngine/flask_code.tar  . && tar xvf flask_code.tar

python3 app.py
```

### Abra o menu Navegação e selecione Security > Web Security Scanner

## Depois de obter pontuação, execute apenas os comandos abaixo.


***Retorne à janela SSH conectada à instância de VM.***

***Pare o aplicativo em execução pressionando CTRL + C.***
***Cole o código abaixo e pressione ENTER.***

```bash
cat > app.py <<EOF_END
import flask
app = flask.Flask(__name__)
input_string = ""

html_escape_table = {
  "&": "&amp;",
  '"': "&quot;",
  "'": "&apos;",
  ">": "&gt;",
  "<": "&lt;",
  }

@app.route('/', methods=["GET", "POST"])
def input():
  global input_string
  if flask.request.method == "GET":
    return flask.render_template("input.html")
  else:
    input_string = flask.request.form.get("input")
    return flask.redirect("output")


@app.route('/output')
def output():
  output_string = "".join([html_escape_table.get(c, c) for c in input_string])
#  output_string = input_string
  return flask.render_template("output.html", output=output_string)

if __name__ == '__main__':
  app.run(host='0.0.0.0', port=8080)
EOF_END


python3 app.py
```


### Parabéns, atividade concluída !!!
