---
layout: post
title: Encontre imagens na Cache do Google Chrome (E Qualquer outro arquivo!)
lang: pt_BR
description: Recupere imagens dos arquivos de cache do chrome e de outro arquivo qualquer!
tags: image chrome jpg jpeg png python carving hacker file find
comments: true
--- 

Boa noite,

Recentemente eu deletei algumas imagens do meu computador e ao tentar acessar as mesmas, os links estavam indisponiveis. Decidi tentar utilizar a cache do navegador. A url antiga para acessar a cache seria `chrome://cache` porém isso não funciona mais nas versões recentes. Para encontrar seus arquivos acesse `/home/USER/.cache/google-chrome/Default/Cache/`. 

Se você abrir o arquivo de cache, você verá que ele não é um arquivo legível diretamente, o mesmo encontra-se em formato binário com outras informações, como URL, headers, http status e outros. Poderiamos olhar o código fonte do Google Chrome para entender como o mesmo gera esses dados e ler os mesmos. Sendo sincero, eu fui preguicoso. [Chrome cache storage](https://github.com/chromium/chromium/tree/f18e79d901f56154f80eea1e2218544285e62623/content/browser/cache_storage)

Por que não escanear os arquivos pela sequencia binaria de um JPEG? Para isso precisarimos aprender como encontrar o inicio e o fim de uma imagem. Da especificação do JPG temos:
- bytes 0xFF, 0xD8 indicam inicio da imagem
- bytes 0xFF, 0xD9 indicam fim da imagem.

Certo, então como encontra estes bytes usando python?

Abra o arquivo como binário e verifique se existe as marcações JFIF ou EXIF. (Removendo arquivos que não precisamos verificar)
```
f = open(filepath, 'rb')

data = f.read()
if 'JFIF' not in data and 'Exif' not in data:
	return
```

Agora vamos iterar sobre todos os bytes, tentando encontrar a sequencia específica. Para fazer isso teremos uma váriavel `prev` a qual vai salvar o valor do byte anterior, pos que vai salvar a posição que estamos, um array para salvar todos os SOI e um para salvar todos os EOI. No caso se o char anterior for FF verificamos se o atual é 0xD8, se sim, adicionamos a SOI, se for D9 adicionamos a EOI.

```
prev = None
soi = []
eoi = []
pos = 0
for b in data:
	if prev is None:
		prev = b
		pos = pos + 1
		continue
	if prev == chr(0xFF):
		if b == chr(0xD8):
			soi.append(pos-1)
		elif b == chr(0xD9):
			eoi.append(pos-1)
	prev = b
	pos = pos + 1
```

Agora podemos selecionar o SOI e o EOI e salvar. A única mágica que estaremos fazendo é pegar o primeiro SOI e o último SOI ou EOI, dependendo qual é maior.
```
path, filename = os.path.split(filepath)
file = open('{}/{}-{}.jpg'.format(OUTPUT_FOLDER, filename, 0), 'wb')
m1 = soi[0]
m2 = soi[-1] if soi[-1] > eoi[-1] else eoi[-1]
file.write(data[m1:m2])

file.close()

print(filename, "SOI", soi, len(soi))
print(filename, "EOI", eoi, len(eoi))
```
Esse código vai salvar somente uma imagem, mas você poderia ter um arquivo com multiplas imagens e salvar multiplos arquivos iterando sobre os EOI/SOI.

Seria isso um tipo de file carving?

Espero que seja util,
Matheus


Pegue este script, crie uma pasta OUTPUT_FOLDER e rode `python yourfile.py filetocheck`, essa versão teoricamente consegue extrair multiplos imagens de um arquivo.

```
import os
import glob
import sys

OUTPUT_FOLDER = "output-this2"


def save_file(data, path, filename, count, eoi, soi):
	file = open('{}/{}-{}.jpg'.format(OUTPUT_FOLDER, filename, count), 'wb')
	m1 = soi[0]
	m2 = soi[-1] if soi[-1] > eoi[-1] else eoi[-1]
	file.write(data[m1:m2])
	file.close()

def extract(filepath):
	count = 0
	f = open(filepath, 'rb')

	data = f.read()
	if 'JFIF' not in data and 'Exif' not in data:
		return

	path, filename = os.path.split(filepath)

	old_soi = []
	old_eoi = []
	prev = None
	soi = []
	eoi = []
	eoi_found = False
	pos = 0
	for b in data:
		if prev is None:
			prev = b
			pos = pos + 1
			continue
		if prev == chr(0xFF):
			if b == chr(0xD8):
				if eoi_found:
					save_file(data, path, filename, count, eoi, soi)
					old_soi = old_soi + soi
					old_eoi = old_eoi + eoi
					soi = []
					eoi = []
					count = count + 1
					eoi_found = False
				soi.append(pos-1)
			elif b == chr(0xD9):
				eoi.append(pos-1)
				eoi_found = True
		prev = b
		pos = pos + 1

	save_file(data, path, filename, count, eoi, soi)
	print(filename, "SOI", soi, len(old_soi))
	print(filename, "EOI", eoi, len(old_eoi))

def main():
	if len(sys.argv) < 2:
		sys.exit(1)

	extract(sys.argv[1])

if __name__=="__main__":
	main()
```

Reference:
[https://stackoverflow.com/questions/4585527/detect-eof-for-jpg-images](https://stackoverflow.com/questions/4585527/detect-eof-for-jpg-images)
