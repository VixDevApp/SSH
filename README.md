#!/bin/bash
clear
#--------@ALFAINTERNET-----------#
source ShellBot.sh
touch lista
[[ -z $1 ]] && {
    clear && echo "INFORME O TOKEN" && return 0
}
[[ ! -e RESET ]] && touch RESET
api_bot=$1
ShellBot.init --token "$api_bot" --monitor --flush
ShellBot.username

# - Funcao menu
menu() {
    local msg
        msg="=×=×=×=×=×=×=×=×=×=×=×=×=×=\n"
        msg+="<b>🤖 OLA SEJA BEM VINDO(a) 🤖</b>\n"
        msg+="=×=×=×=×=×=×=×=×=×=×=×=×=×=\n\n"
        msg+="Gere seu teste agora mesmo!"
        ShellBot.sendMessage --chat_id ${message_chat_id[$id]} \
        --text "$(echo -e $msg)" \
        --reply_markup "$keyboard1" \
        --parse_mode html
        return 0
}

# - funcao criar ssh
criarteste() {
    [[ $(grep -wc ${callback_query_from_id} lista) != '0' ]] && {
      ShellBot.sendMessage --chat_id ${callback_query_message_chat_id} \
        --text "⚠️ VC JA CRIOU SSH HOJE ⚠️"
      return 0
    }
    usuario=$(echo TESTE$(( RANDOM% + 999 )))
    senha=$((RANDOM% + 999999))
    limite='1'
    tempo='1'
    tuserdate=$(date '+%C%y/%m/%d' -d " +1 days")
    useradd -M -N -s /bin/false $usuario -e $tuserdate > /dev/null 2>&1
    (echo "$senha";echo "$senha") | passwd $usuario > /dev/null 2>&1
    echo "$senha" > /etc/SSHPlus/senha/$usuario
    echo "$usuario $limite" >> /root/usuarios.db
    echo "#!/bin/bash
pkill -f "$usuario"
userdel --force $usuario
grep -v ^$usuario[[:space:]] /root/usuarios.db > /tmp/ph ; cat /tmp/ph > /root/usuarios.db
rm /etc/SSHPlus/senha/$usuario > /dev/null 2>&1
rm -rf /etc/SSHPlus/userteste/$usuario.sh" > /etc/SSHPlus/userteste/$usuario.sh
    chmod +x /etc/SSHPlus/userteste/$usuario.sh
    at -f /etc/SSHPlus/userteste/$usuario.sh now + $tempo hour > /dev/null 2>&1
    echo ${callback_query_from_id} >> lista
    # - ENVIA O SSH
    ShellBot.sendMessage --chat_id ${callback_query_message_chat_id} \
    --text "$(echo -e "✅ <b>Criado com sucesso</b> ✅\n\nUSUARIO: <code>$usuario</code>\nSENHA: <code>$senha</code>\n\n⏳ Expira em: $tempo Hora")" \
    --parse_mode html
    return 0
}

#enviar app
enviarapp() {
    ShellBot.answerCallbackQuery --callback_query_id ${callback_query_id[$id]} \
        --text "♻️ ENVIANDO APLICATIVO, AGUARDE..."
    ShellBot.sendDocument --chat_id ${callback_query_message_chat_id} \
        --document "@/root/app.apk" \
    return 0


    }

#informacoes usuario
infouser () {
	ShellBot.sendMessage --chat_id ${message_chat_id[$id]} \
	--text "$(echo -e "Nome:  ${message_from_first_name[$(ShellBot.ListUpdates)]}\nUser: @${message_from_username[$(ShellBot.ListUpdates)]:-null}")\nID: ${message_from_id[$(ShellBot.ListUpdates)]} " \
	--parse_mode html
	return 0
}

unset botao1
botao1=''
ShellBot.InlineKeyboardButton --button 'botao1' --line 1 --text '♻️ GERAR TESTE ♻️' --callback_data 'gerarssh'
ShellBot.InlineKeyboardButton --button 'botao1' --line 3 --text '🔰 BAIXAR APLICATIVO 🔰' --callback_data 'appenviar'
ShellBot.regHandleFunction --function criarteste --callback_data gerarssh
ShellBot.regHandleFunction --function enviarapp --callback_data appenviar
unset keyboard1
keyboard1="$(ShellBot.InlineKeyboardMarkup -b 'botao1')"
while :; do
   [[ "$(date +%d)" != "$(cat RESET)" ]] && {
   	echo $(date +%d) > RESET
   	echo ' ' > lista
   }
  ShellBot.getUpdates --limit 100 --offset $(ShellBot.OffsetNext) --timeout 24
  for id in $(ShellBot.ListUpdates); do
    (
      ShellBot.watchHandle --callback_data ${callback_query_data[$id]}
      comando=(${message_text[$id]})
      [[ "${comando[0]}" = "/menu"  || "${comando[0]}" = "/start" ]] && menu
      [[ "${comando[0]}" = "/id"  ]] && infouser
    ) &
  done
done
