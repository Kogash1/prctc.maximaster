<template>
  <div class="order-form">
    <form @submit.prevent="submitForm">
      <div class="input-group">
        <input 
          type="text" 
          v-model="form.name" 
          placeholder="ФИО"
          required
        />
      </div>

      <div class="input-group">
        <input 
          type="text" 
          v-model="form.phone" 
          placeholder="Телефон"
          required
        />
      </div>

      <div class="input-group">
        <input 
          type="email" 
          v-model="form.email" 
          placeholder="Email"
        />
      </div>

      <div id="map" class="map"></div>

      <div v-if="form.coords" class="coords">
        Координаты: {{ form.coords[0].toFixed(5) }}, {{ form.coords[1].toFixed(5) }}
      </div>

      <div class="input-group">
        <textarea 
          v-model="form.comment" 
          placeholder="Комментарий к заказу (макс. 500 символов)"
          maxlength="500" 
          rows="6"
        ></textarea>
      </div>

      <button type="submit" class="submit-btn">Отправить</button>

      <div v-if="message" :class="['message', isError ? 'error' : 'success']">
        {{ message }}
      </div>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      form: {
        name: '',
        phone: '',
        email: '',
        comment: '',
        coords: null,
      },
      message: '',
      isError: false,
      map: null,
      placemark: null,
    };
  },
  mounted() {
    if (!window.ymaps) {
      const script = document.createElement('script');
      script.src = 'https://api-maps.yandex.ru/2.1/?apikey=f69d2557-72b2-4a26-9bff-5b315b9ccbb8&lang=ru_RU'; 
      script.onload = this.initMap;
      document.head.appendChild(script);
    } else {
      this.initMap();
    }
  },
  methods: {
    initMap() {
      window.ymaps.ready(() => {
        this.map = new window.ymaps.Map('map', {
          center: [54.20, 37.57],
          zoom: 11,
        });

        this.map.events.add('click', (e) => {
          const coords = e.get('coords');
          this.form.coords = coords;

          const content = `Координаты: ${coords[0].toFixed(5)}, ${coords[1].toFixed(5)}`;

          if (this.placemark) {
            this.map.geoObjects.remove(this.placemark);
            this.placemark = null;
          }

          this.placemark = new window.ymaps.Placemark(
            coords,
            {
              balloonContent: content,
              hintContent: 'Точка доставки',
            },
            {
              draggable: true,
            }
          );

          this.placemark.events.add('dragend', () => {
            const newCoords = this.placemark.geometry.getCoordinates();
            this.form.coords = newCoords;
          });

          this.map.geoObjects.add(this.placemark);
        });
      });
    },
    submitForm() {
      const errors = [];

      if (!this.form.name.trim()) {
        errors.push('Не заполнено поле ФИО');
      }

      if (!this.form.phone.trim()) {
        errors.push('Не заполнено поле Телефон');
      } else if (!/^\d+$/.test(this.form.phone)) {
        errors.push('Телефон должен содержать только цифры');
      }

      if (this.form.email && !this.form.email.includes('@')) {
        errors.push('Email должен содержать символ "@"');
      }

      if (!this.form.coords) {
        errors.push('Не отмечен адрес доставки на карте');
      }

      if (errors.length > 0) {
        this.message = errors.join(', ');
        this.isError = true;
      } else {
        this.message = 'Заказ оформлен!';
        this.isError = false;
      }
    },
  },
};
</script>

<style scoped>
.order-form {
  max-width: 650px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

form {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.input-group {
  position: relative;
}

input,
textarea {
  width: 500px;
  height: 30px;
  padding: 10px;
  border: 1px solid #black;
  font-size: 16px;
  background-color: #fff;
}

input::placeholder,
textarea::placeholder {
  color: #d3d1d1;
  opacity: 1;
}

textarea {
  resize: none;
  height: 190px;
  font-family: inherit;
  border: 2px solid #black;
  padding: 10px;
}

.map {
  width: 100%;
  height: 240px;
  margin: 10px 0 20px;
  border: 2px solid #b0b0b0ff;
}

.coords {
  font-size: 14px;
  color: #666;
  text-align: center;
  margin: -30px 0 1px;
}

.submit-btn {
  width: 180px;
  height: 30px;
  background-color: #e6e6e6;
  color: black;
  border-radius: 4px;
  font-size: 17px;
  cursor: pointer;
  align-self: center;
}

.message {
  font-size: 16px;
  border-radius: 4px;
  text-align: center;
  margin-top: -10px;
  font-weight: 700;
}

.error {
  background-color: #ffffffff;
  color: #d32f2f;
}

.success {
  background-color: #ffffffff;
  color: #2e7d32;
}
</style>