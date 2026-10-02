const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const P = require('pino')
const readline = require('readline')

const rl = readline.createInterface({ input: process.stdin, output: process.stdout })
const question = (text) => new Promise((resolve) => rl.question(text, resolve))

async function startPairing() {
  const { state, saveCreds } = await useMultiFileAuthState('./auth_code_session')
  
  const sock = makeWASocket({
    auth: state,
    logger: P({ level: 'silent' }),
    printQRInTerminal: false,
    browser: ["Ubuntu", "Chrome", "20.0.04"]
  })

  sock.ev.on('creds.update', saveCreds)

  // لو لسه مش مسجل، اطلب كود
  if (!sock.authState.creds.registered) {
    console.log('\n╔════════════════════════════════════╗')
    console.log('║  Kingdom Meteors - ربط بالكود 8    ║')
    console.log('╚════════════════════════════════════╝\n')
    
    let phoneNumber = await question('📱 اكتب رقمك بالكود الدولي بدون + وبدون مسافات\nمثال: 201012345678\n> ')
    phoneNumber = phoneNumber.trim().replace(/\D/g, '') // يشيل أي حاجة مش رقم
    
    if (!phoneNumber) {
      console.log('❌ لازم تكتب رقم')
      process.exit(1)
    }
    
    console.log(`\n⏳ بطلب كود 8 أرقام للرقم ${phoneNumber}...\n`)
    
    try {
      // ده اللي بيجيب كود 8 أرقام
      let code = await sock.requestPairingCode(phoneNumber)
      code = code?.match(/.{1,4}/g)?.join('-') || code
      
      console.log('╔════════════════════════════════════╗')
      console.log(`║  ✅ كود الربط: ${code}          ║`)
      console.log('╚════════════════════════════════════╝')
      console.log('\n📲 خطوات الربط:')
      console.log('1. افتح واتساب على تليفونك')
      console.log('2. الإعدادات > الأجهزة المرتبطة')
      console.log('3. ربط جهاز > ربط برقم هاتف')
      console.log(`4. اكتب الكود ده: ${code}\n`)
      console.log('⏳ مستني تربط... سيب الشاشة مفتوحة\n')
      
    } catch (error) {
      console.log('❌ خطأ:', error.message)
      console.log('تأكد ان:')
      console.log('- الرقم بالكود الدولي (مثال 2010...)')
      console.log('- النت شغال')
      console.log('- مانزلتش Baileys: npm install @whiskeysockets/baileys')
      process.exit(1)
    }
  }

  sock.ev.on('connection.update', async (update) => {
    const { connection } = update
    if (connection === 'open') {
      console.log('\n✅✅✅ تم الربط بنجاح! Kingdom Meteors شغال الآن 🔥')
      console.log('تقدر دلوقتي تشغل البوت الرئيسي: node pairing-bot.js أو python bot.py')
      rl.close()
    }
  })
}

startPairing()
