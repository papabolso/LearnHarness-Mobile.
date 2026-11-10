# Build deste fork pelo celular (GitHub Actions)

Este fork compila o próprio APK na nuvem, a partir do código-fonte, sem usar o APK publicado pelo
autor da adaptação. Nada aqui muda o comportamento do app, os prompts originais ou as licenças.

## O que o workflow `Build APK` faz

Roda a cada push na `main` e também manualmente (Actions → Build APK → Run workflow).

1. Confere o `gradle-wrapper.jar` contra os checksums oficiais do Gradle.
2. Regenera o bundle do agente (`android/app/src/main/assets/core/`) com `npm ci`, `npm run build`
   e `npm run test` em `android/tools/original-core`, e compara com o que está commitado. O
   resumo do run mostra **Idêntico** ou **Diferente**; se diferente, o APK usa o recém-compilado.
3. Compara `src/` com o LearnHarness original na revisão de base (`UPSTREAM_REV` no workflow).
4. Roda `:app:testDebugUnitTest :app:assembleDebug` com JDK 17 e Android SDK 36.
5. Mostra no resumo o SHA-256 do APK, o certificado, as permissões finais e os endereços de rede.
6. Publica o artifact **learnharness-mobile-debug** (só quando assinado com a chave do fork).

## Chave de assinatura (uma vez só)

O Android só instala uma versão nova por cima (sem perder dados) se ela vier assinada com a
mesma chave. Por isso o fork usa uma chave fixa, guardada nos secrets do repositório. Ela nunca
é commitada (`*.p12`, `*.jks` e `*.keystore` estão no `.gitignore`).

Para criar um secret pelo navegador do celular: abra o repositório no github.com (se a página de
Settings não aparecer, ative **Versão para computador** no menu do navegador) → **Settings** →
**Secrets and variables** → **Actions** → **New repository secret**. O app GitHub para Android
não edita secrets.

1. Crie o secret `ANDROID_KEYSTORE_PASSWORD` com uma senha longa (16+ caracteres, de preferência
   gerada pelo gerenciador de senhas). Guarde-a também no gerenciador.
2. Em **Actions** → **Gerar chave de assinatura (uma vez)** → **Run workflow**.
3. Quando terminar, abra o run e baixe o artifact **chave-assinatura**. Dentro do zip, abra
   `ANDROID_KEYSTORE_BASE64.txt`, selecione tudo e copie.
4. Crie o secret `ANDROID_KEYSTORE_BASE64` colando esse texto.
5. Guarde o zip num lugar seguro fora do GitHub (é o backup da chave; sem ele, perder o
   repositório significa desinstalar o app para trocar de chave). Depois apague o artifact no run
   (ícone de lixeira). Ele expira sozinho em 1 dia.

O keystore é protegido pela senha do passo 1. O repositório é público e qualquer pessoa logada
pode baixar artifacts, então a senha forte é o que protege o arquivo nesse intervalo.

## Baixar e instalar o APK

**Actions** → **Build APK** → o run mais recente com ✅ → seção **Artifacts** no fim da página →
**learnharness-mobile-debug**. O GitHub entrega um `.zip`: abra no app Arquivos, extraia e toque em
`learnharness-mobile-debug.apk`. Na primeira vez, o Android pede para permitir "instalar apps
desconhecidos" para o navegador ou o app Arquivos.

Se você já tiver instalado o APK do autor original, desinstale antes: a assinatura é outra e o
Android não deixa atualizar por cima.

## Ao atualizar a base

Ao trazer mudanças do upstream ou da adaptação, atualize `UPSTREAM_REV` no workflow e confira o
resumo do run: bundle idêntico e `src/` idêntico (ou só as diferenças que você espera).
