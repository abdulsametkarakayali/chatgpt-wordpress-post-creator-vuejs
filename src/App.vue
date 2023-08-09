<template>
  
  <div class="container mt-5">


    <div class="config-form">
    <h2>API ve WordPress Ayarları</h2>
    <div>
      <label for="openai-api-key">OpenAI API Key:</label>
      <input v-model="openaiApiKey" class="form-control" type="text" id="openai-api-key">
    </div>
    <div>
      <label for="wordpress-url">WordPress URL:</label>
      <input v-model="wordpressUrl" class="form-control" type="text" id="wordpress-url">
    </div>
    <div>
      <label for="wordpress-username">WordPress Kullanıcı Adı:</label>
      <input v-model="wordpressUsername" class="form-control" type="text" id="wordpress-username">
    </div>
    <div>
      <label for="wordpress-password">WordPress Şifre:</label>
      <input v-model="wordpressPassword" class="form-control" type="password" id="wordpress-password">
    </div>
  </div>


    <div v-if="icerikOlusturuldu" class="alert alert-success" role="alert">
        Başarılı!
      </div>
      <div class="mt-4" v-if="loading">
      <div class="progress">
        <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" aria-valuenow="100" aria-valuemin="0" aria-valuemax="100" style="width: 100%"></div>
      </div>
    </div>
    <h1 class="mb-4"> Seo İçerik Oluşturucu</h1>
    <div class="form-group">
      <label class="mb-2">Konu Başlığı</label>
      <input v-model="konuBasligi" class="form-control" placeholder="Konu Başlığı">
    </div>
    <button @click="icerikOlustur" :disabled="islemBasladi" class="mt-2 btn btn-primary">İçerik Oluştur</button>
    <div class="icerik mt-4" v-if="olusturulanIcerik">
      <p v-if="loading">İçerik oluşturuluyor...</p>
      <h2>Oluşturulan İçerik</h2>
      <p>{{ olusturulanIcerik }}</p>
      <button @click="icerigiPaylas" :disabled="islemBasladi" class="btn btn-success">İçeriği Paylaş</button>
    </div>

  </div>

</template>




<script>
import axios from 'axios';

export default {
  data() {
    return {
      openaiApiKey:'',
      wordpressUrl:'',
      wordpressUsername:'',
      wordpressPassword:'',
      icerikOlusturuldu: false,
      konuBasligi: '',
      islemBasladi: false,
      olusturulanIcerik: '',
      loading: false,
    };
  },
  methods: {
        async icerikOlustur() {
          this.loading = true; 
          this.islemBasladi = true;
          //const apiKey = process.env.VUE_APP_OPENAI_API_KEY;
          console.log(process.env.VUE_APP_OPENAI_API_KEY)
          try {
            const response = await axios.post(
            'https://api.openai.com/v1/chat/completions',
            {
              model: "gpt-3.5-turbo",
              messages: [{"role": "user", "content": this.konuBasligi}] ,
              max_tokens: 700,
            },
            {
              headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${this.openaiApiKey}`,
              },
            }
          );
          this.olusturulanIcerik = response.data.choices[0].message.content;
          this.icerikOlusturuldu = true;
          this.islemBasladi = false;
          } catch (error) {
            console.error('İçerik oluşturulurken hata:', error);
          } finally {
            this.loading = false;
            this.islemBasladi= false;
          }
        
        },
     async icerigiPaylas() {
      this.loading = true; 
      this.islemBasladi = true;
      this.icerikOlusturuldu = false;
      const wordpressUrl = this.wordpressUrl;
      const username = this.wordpressUsername;
      const password = this.wordpressPassword;


        const auth = 'Basic ' + btoa(username + ':' + password);
        try {
          const response = await axios.post(
          `${wordpressUrl}/wp-json/wp/v2/posts`,
          {
            title: this.konuBasligi,
            content: this.olusturulanIcerik,
            status: 'publish',
          },
          {
            headers: {
              'Content-Type': 'application/json',
              'Authorization': auth,
            },
          }
        );
        this.icerikOlusturuldu = true;
        console.log('Yazı paylaşıldı:', response.data);
        } catch (error) {
          console.error('İçerik oluşturulurken hata:', error);
        }finally {
            this.loading = false; 
            this.islemBasladi= false;
          }
      },
  },
};
</script>
<style>
/* İhtiyaca göre özelleştirmeler yapılabilir */
.icerik {
  border-top: 1px solid #ccc;
  padding-top: 20px;
}
</style>