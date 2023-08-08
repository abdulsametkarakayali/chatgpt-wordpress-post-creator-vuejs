<template>
  <div class="container mt-5">
    <h1 class="mb-4">İçerik Oluşturucu</h1>
    <div class="form-group">
      <label>Konu Başlığı</label>
      <input v-model="konuBasligi" class="form-control" placeholder="Konu Başlığı">
    </div>
    <button @click="icerikOlustur" class="btn btn-primary">İçerik Oluştur</button>
    <div class="icerik mt-4" v-if="olusturulanIcerik">
      <p v-if="loading">İçerik oluşturuluyor...</p>
      <h2>Oluşturulan İçerik</h2>
      <p>{{ olusturulanIcerik }}</p>
      <button @click="icerigiPaylas" class="btn btn-success">İçeriği Paylaş</button>
    </div>
    <div class="mt-4" v-if="loading">
      <div class="progress">
        <div class="progress-bar progress-bar-striped progress-bar-animated" role="progressbar" aria-valuenow="100" aria-valuemin="0" aria-valuemax="100" style="width: 100%"></div>
      </div>
    </div>
  </div>

</template>




<script>
import axios from 'axios';

export default {
  data() {
    return {
      konuBasligi: '',
      olusturulanIcerik: '',
      loading: false,
    };
  },
  methods: {
        async icerikOlustur() {
          this.loading = true; 
          const apiKey = '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';
          try {
            const response = await axios.post(
            'https://api.openai.com/v1/chat/completions',
            {
              model: "gpt-3.5-turbo",
              messages: [{"role": "user", "content": this.konuBasligi}] ,
              max_tokens: 10,
            },
            {
              headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${apiKey}`,
              },
            }
          );
          this.olusturulanIcerik = response.data.choices[0].message.content;
          } catch (error) {
            console.error('İçerik oluşturulurken hata:', error);
          } finally {
            this.loading = false;
          }
        
        },
     async icerigiPaylas() {
      this.loading = true; 
        const wordpressUrl = 'https://www.01adana.com.tr';
        const username = 'admin';
        const password = 'rMtB e9r0 XqED KFHI WWaM chD0';

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
        console.log('Yazı paylaşıldı:', response.data);
        } catch (error) {
          console.error('İçerik oluşturulurken hata:', error);
        }finally {
            this.loading = false; 
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