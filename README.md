// اسم الملف في الريبو: code.js
// شغله: node code.js
// هيطلع كود 8 أرقام

const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const P = require('pino')
const readline = require('readline')

const rl = readline.createInterface({ input: process.stdin, output: process.stdout })
const q = (t) => new Promise(r => rl.question(t, r))

async function start() {
  const { state, saveCreds } = await useMultiFileAuthState('./auth')
  const sock = makeWASocket({
    auth: state,
    logger: P({ level: 'silent' }),
    printQRInTerminal: false,
    browser: ["Ubuntu", "Chrome", "20.0"]
  })
  sock.ev.on('creds.update', saveCreds)

  if (!sock.authState.creds.registered) {
    console.log('\n=== Kingdom Meteors Pairing ===')
    let num = await q('رقمك بالكود الدولي مثال 201002707721: ')
    num = num.replace(/[^0-9]/g,'')
    console.log(`\nجاري طلب كود للرقم ${num}...`)
    try {
      let code = await sock.requestPairingCode(num)
      code = code.match(/.{1,4}/g).join('-')
      console.log('\n╔════════════════════╗')
      console.log(`║ كودك: ${code}     ║`)
      console.log('╚════════════════════╝')
      console.log('\nواتساب > الأجهزة المرتبطة > ربط برقم هاتف > اكتب الكود')
    } catch(e){ console.log('خطأ', e.message) }
  }
  sock.ev.on('connection.update', ({connection})=>{
    if(connection==='open'){ console.log('\n✅ اتربط بنجاح!'); rl.close() }
  })
}
start()
