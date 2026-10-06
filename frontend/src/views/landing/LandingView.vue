<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue';
import { useAuthStore } from '@/stores/auth.store';
import heroImg from '@/assets/images/landing-hero.jpg';
import storyImg from '@/assets/images/landing-story.jpg';

const auth = useAuthStore();
const roomTypes = [
  { value: 'garden', label: 'Garden Room', detail: 'Vườn riêng · 42 m²', image: 'https://images.unsplash.com/photo-1618221195710-dd6b41faaea6?auto=format&fit=crop&w=900&q=80' },
  { value: 'ocean', label: 'Ocean View Room', detail: 'Hướng biển · 48 m²', image: 'https://images.unsplash.com/photo-1590490360182-c33d57733427?auto=format&fit=crop&w=900&q=80' },
  { value: 'suite', label: 'Lumina Suite', detail: 'Phòng khách riêng · 72 m²', image: 'https://images.unsplash.com/photo-1611892440504-42a792e24d32?auto=format&fit=crop&w=900&q=80' },
  { value: 'villa', label: 'Private Pool Villa', detail: 'Hồ bơi riêng · 110 m²', image: 'https://images.unsplash.com/photo-1566665797739-1674de7a421a?auto=format&fit=crop&w=900&q=80' },
];

const isoDate = (date: Date) => {
  const local = new Date(date.getTime() - date.getTimezoneOffset() * 60_000);
  return local.toISOString().slice(0, 10);
};
const today = new Date();
const tomorrow = new Date();
tomorrow.setDate(tomorrow.getDate() + 1);

const form = reactive({
  fullName: '',
  phone: '',
  identityNumber: '',
  email: '',
  checkIn: isoDate(today),
  checkOut: isoDate(tomorrow),
  checkInTime: '14:00',
  checkOutTime: '12:00',
  guests: 2,
  roomType: '',
  privacyConsent: false,
  note: '',
});

const feedback = ref('');
const feedbackIsError = ref(false);
const bookingAttempted = ref(false);
const checkoutMin = computed(() => {
  const date = form.checkIn ? new Date(`${form.checkIn}T00:00:00`) : new Date(tomorrow);
  date.setDate(date.getDate() + (form.checkIn ? 1 : 0));
  return isoDate(date);
});
const nights = computed(() => {
  if (!form.checkIn || !form.checkOut) return 0;
  const start = new Date(`${form.checkIn}T00:00:00`);
  const end = new Date(`${form.checkOut}T00:00:00`);
  return Math.max(0, Math.round((end.getTime() - start.getTime()) / 86_400_000));
});
const selectedRoom = computed(() => roomTypes.find((room) => room.value === form.roomType));
const bookingErrors = computed(() => ({
  checkIn: !form.checkIn ? 'Vui lòng chọn ngày nhận phòng.' : '',
  checkOut: nights.value < 1 ? 'Ngày trả phòng cần sau ngày nhận phòng ít nhất một đêm.' : '',
  roomType: !form.roomType ? 'Vui lòng chọn một hạng phòng.' : '',
  fullName: form.fullName.trim().length < 2 ? 'Vui lòng nhập họ tên từ 2 ký tự.' : '',
  phone: !/^[0-9+ ()-]{9,16}$/.test(form.phone.trim()) ? 'Vui lòng nhập số điện thoại hợp lệ.' : '',
  identityNumber: !/^[A-Za-z0-9-]{9,20}$/.test(form.identityNumber.trim()) ? 'Nhập CCCD hoặc hộ chiếu từ 9 đến 20 ký tự.' : '',
  email: form.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email.trim()) ? 'Email chưa đúng định dạng.' : '',
  privacyConsent: !form.privacyConsent ? 'Vui lòng xác nhận đồng ý sử dụng thông tin cho yêu cầu này.' : '',
}));

onMounted(() => {
  const user = auth.user as (NonNullable<typeof auth.user> & { full_name?: string; phone_number?: string }) | null;
  if (!user) return;
  form.fullName = user.fullName || user.full_name || '';
  form.email = user.email || '';
  form.phone = user.phoneNumber || user.phone_number || '';
});

function updateCheckoutMinimum() {
  if (!form.checkIn) return;
  const minimum = new Date(`${form.checkIn}T00:00:00`);
  minimum.setDate(minimum.getDate() + 1);
  const minDate = isoDate(minimum);
  if (!form.checkOut || form.checkOut < minDate) form.checkOut = minDate;
}

function selectRoom(roomType: string) {
  form.roomType = roomType;
}

function submitBooking() {
  feedback.value = '';
  feedbackIsError.value = false;
  bookingAttempted.value = true;
  if (Object.values(bookingErrors.value).some(Boolean)) {
    feedbackIsError.value = true;
    feedback.value = 'Vui lòng kiểm tra các mục được đánh dấu bên dưới.';
    return;
  }

  feedback.value = `Đã kiểm tra thông tin cho ${selectedRoom.value?.label} · ${nights.value} đêm. Backend hiện chưa có API đặt phòng nên yêu cầu chưa được gửi.`;
}
</script>

<template>
  <div class="lumina-page">
    <header class="topbar">
      <a class="brand" href="#top" aria-label="Lumina, đầu trang">LUMINA<span>®</span></a>
      <nav class="top-nav" aria-label="Điều hướng chính">
        <a href="#stay">Kỳ nghỉ</a>
        <a href="#booking">Đặt phòng</a>
        <a href="#footer">Liên hệ</a>
      </nav>
      <RouterLink v-if="!auth.isAuthenticated()" class="account-link" to="/login">Đăng nhập <span>↗</span></RouterLink>
      <span v-else class="account-link signed-in">{{ auth.user?.fullName || 'Tài khoản của tôi' }}</span>
    </header>

    <main id="top">
      <section class="hero" aria-labelledby="hero-title">
        <div class="hero-photo" :style="{ backgroundImage: `url(${heroImg})` }" role="img" aria-label="Bờ biển Bãi Xép, Phú Yên, miền Trung Việt Nam" />
        <div class="hero-overlay" />
        <div class="hero-content">
          <p class="kicker">PHÚ YÊN · MIỀN TRUNG VIỆT NAM</p>
          <h1 id="hero-title">Trở về với<br /><em>những điều giản dị.</em></h1>
          <p class="hero-description">Một chốn nghỉ yên bình để sống chậm,<br class="desktop-only" /> hít thở sâu và cảm nhận nhiều hơn.</p>
          <a class="hero-link" href="#booking">Khám phá kỳ nghỉ <span>↓</span></a>
        </div>
        <div class="hero-annotation" aria-label="Gợi ý trải nghiệm: nghỉ chậm, sống sâu">
          <span class="annotation-spark" aria-hidden="true">✳</span>
          <span><small>MỘT NHỊP SỐNG KHÁC</small><strong>Nghỉ chậm, sống sâu</strong></span>
          <span class="annotation-index">01 — 03</span>
        </div>
        <span class="hero-side-note" aria-hidden="true">PHÚ YÊN · 2026</span>
        <div class="hero-meta"><span>BÃI XÉP · PHÚ YÊN</span><span>DUYÊN HẢI MIỀN TRUNG</span></div>
      </section>

      <section id="booking" class="booking-section">
        <div class="booking-heading">
          <p class="section-kicker">KỲ NGHỈ CỦA BẠN</p>
          <h2>Hãy để chúng tôi<br /><em>lo phần còn lại.</em></h2>
          <p>Chia sẻ một vài thông tin để chúng tôi chuẩn bị đón tiếp bạn chu đáo.</p>
          <p class="required-note"><b>*</b> Các mục có dấu sao là thông tin bắt buộc.</p>
          <div class="guest-mode">
            <span class="mode-dot" />
            <span v-if="auth.isAuthenticated()">Đang đặt với hồ sơ <strong>{{ auth.user?.fullName }}</strong></span>
            <span v-else>Đặt phòng với tư cách <strong>khách</strong></span>
          </div>
        </div>

        <form class="booking-form" novalidate @submit.prevent="submitBooking">
          <div class="form-section-label"><span>01</span> THÔNG TIN LƯU TRÚ</div>
          <div class="form-grid stay-fields">
            <label class="field" :class="{ 'field-invalid': bookingAttempted && bookingErrors.checkIn }" for="check-in"><span>Ngày nhận phòng <b>*</b></span><input id="check-in" v-model="form.checkIn" type="date" :min="isoDate(today)" required aria-describedby="check-in-error" @change="updateCheckoutMinimum" /><small v-if="bookingAttempted && bookingErrors.checkIn" id="check-in-error" class="field-error">{{ bookingErrors.checkIn }}</small></label>
            <label class="field" :class="{ 'field-invalid': bookingAttempted && bookingErrors.checkOut }" for="check-out"><span>Ngày trả phòng <b>*</b></span><input id="check-out" v-model="form.checkOut" type="date" :min="checkoutMin" required aria-describedby="check-out-error" /><small v-if="bookingAttempted && bookingErrors.checkOut" id="check-out-error" class="field-error">{{ bookingErrors.checkOut }}</small></label>
            <label class="field" for="arrival-time"><span>Giờ nhận phòng</span><select id="arrival-time" v-model="form.checkInTime"><option>14:00</option><option>15:00</option><option>16:00</option><option>17:00</option><option>18:00</option><option>19:00</option><option>20:00</option></select></label>
            <label class="field" for="departure-time"><span>Giờ trả phòng</span><select id="departure-time" v-model="form.checkOutTime"><option>08:00</option><option>09:00</option><option>10:00</option><option>11:00</option><option>12:00</option><option>13:00</option></select></label>
            <label class="field" for="guests"><span>Số khách</span><select id="guests" v-model.number="form.guests"><option :value="1">1 khách</option><option :value="2">2 khách</option><option :value="3">3 khách</option><option :value="4">4 khách</option><option :value="5">5 khách</option><option :value="6">6 khách</option></select></label>
            <div class="room-picker-wrap full-field" :class="{ 'room-picker-invalid': bookingAttempted && bookingErrors.roomType }">
              <div class="room-picker-heading">
                <div><span class="room-picker-title">Chọn hạng phòng <b>*</b></span><p>Chọn một không gian phù hợp với kỳ nghỉ của bạn.</p></div>
                <span class="room-picker-count">{{ selectedRoom ? 'ĐÃ CHỌN 1 HẠNG PHÒNG' : 'CHỌN 1 HẠNG PHÒNG' }}</span>
              </div>
              <div class="room-card-grid" role="radiogroup" aria-label="Chọn hạng phòng">
                <button v-for="room in roomTypes" :key="room.value" class="room-card" :class="{ selected: form.roomType === room.value }" type="button" role="radio" :aria-checked="form.roomType === room.value" @click="selectRoom(room.value)">
                  <img class="room-card-image" :src="room.image" :alt="`Ảnh minh họa ${room.label}`" loading="lazy" />
                  <span class="room-card-copy"><span class="room-card-name">{{ room.label }}</span><span class="room-card-detail">{{ room.detail }}</span><span class="room-card-status">Tình trạng xác nhận theo ngày</span></span>
                  <span class="room-card-check" aria-hidden="true">✓</span>
                </button>
              </div>
              <small v-if="bookingAttempted && bookingErrors.roomType" class="field-error room-picker-error">{{ bookingErrors.roomType }}</small>
              <div v-if="selectedRoom" class="selected-room-preview">
                <img :src="selectedRoom.image" :alt="`Ảnh tham khảo ${selectedRoom.label}`" />
                <div><span class="preview-kicker">HẠNG PHÒNG BẠN ĐANG XEM</span><strong>{{ selectedRoom.label }}</strong><span>{{ selectedRoom.detail }}</span><small>Ảnh tham khảo · Hình ảnh thực tế có thể khác.</small></div>
              </div>
              <p class="availability-note"><span aria-hidden="true">✳</span> Tình trạng phòng trống sẽ được xác nhận theo ngày lưu trú bạn chọn.</p>
            </div>
          </div>

          <div class="form-section-label guest-label"><span>02</span> THÔNG TIN KHÁCH ĐẠI DIỆN</div>
          <div class="form-grid contact-fields">
            <label class="field" :class="{ 'field-invalid': bookingAttempted && bookingErrors.fullName }" for="full-name"><span>Họ và tên <b>*</b></span><input id="full-name" v-model.trim="form.fullName" type="text" autocomplete="name" placeholder="Tên trên giấy tờ tuỳ thân" required minlength="2" aria-describedby="full-name-error" /><small v-if="bookingAttempted && bookingErrors.fullName" id="full-name-error" class="field-error">{{ bookingErrors.fullName }}</small></label>
            <label class="field" :class="{ 'field-invalid': bookingAttempted && bookingErrors.phone }" for="phone"><span>Số điện thoại <b>*</b></span><input id="phone" v-model.trim="form.phone" type="tel" autocomplete="tel" placeholder="090 123 4567" required pattern="[0-9+ ()-]{9,16}" aria-describedby="phone-error" /><small v-if="bookingAttempted && bookingErrors.phone" id="phone-error" class="field-error">{{ bookingErrors.phone }}</small></label>
            <label class="field" :class="{ 'field-invalid': bookingAttempted && bookingErrors.identityNumber }" for="identity"><span>Số CCCD / hộ chiếu <b>*</b></span><input id="identity" v-model.trim="form.identityNumber" type="text" placeholder="Số trên giấy tờ tuỳ thân" required minlength="9" maxlength="20" aria-describedby="identity-error" /><small v-if="bookingAttempted && bookingErrors.identityNumber" id="identity-error" class="field-error">{{ bookingErrors.identityNumber }}</small></label>
            <label class="field" :class="{ 'field-invalid': bookingAttempted && bookingErrors.email }" for="email"><span>Email <small>Không bắt buộc</small></span><input id="email" v-model.trim="form.email" type="email" autocomplete="email" placeholder="ban@email.com" aria-describedby="email-error" /><small v-if="bookingAttempted && bookingErrors.email" id="email-error" class="field-error">{{ bookingErrors.email }}</small></label>
            <label class="field full-field" for="note"><span>Ghi chú cho kỳ nghỉ <small>Không bắt buộc</small></span><textarea id="note" v-model.trim="form.note" rows="2" placeholder="Giờ đến dự kiến, yêu cầu đặc biệt..."></textarea></label>
          </div>

          <div class="form-bottom">
            <label class="privacy-consent" :class="{ 'consent-invalid': bookingAttempted && bookingErrors.privacyConsent }">
              <input v-model="form.privacyConsent" type="checkbox" />
              <span class="consent-check" aria-hidden="true">✓</span>
              <span class="consent-copy">Tôi đồng ý để Lumina sử dụng thông tin liên hệ nhằm xử lý yêu cầu lưu trú này. <b>*</b></span>
            </label>
            <button type="submit" class="submit-button">Gửi yêu cầu <span>→</span></button>
          </div>
          <small v-if="bookingAttempted && bookingErrors.privacyConsent" class="field-error consent-error">{{ bookingErrors.privacyConsent }}</small>
          <p v-if="feedback" class="feedback" :class="{ error: feedbackIsError }" role="status" aria-live="polite">{{ feedback }}</p>
        </form>
      </section>

      <section id="stay" class="stay-story">
        <div class="story-image" :style="{ backgroundImage: `url(${storyImg})` }" role="img" aria-label="Kinh thành Huế, di sản văn hóa miền Trung Việt Nam" />
        <div class="story-copy"><p class="section-kicker">DẤU ẤN MIỀN TRUNG</p><h2>Vẻ đẹp của<br /><em>những khoảng lặng.</em></h2><p>Từ sắc xanh yên bình của biển Phú Yên đến nét trầm mặc của kinh thành Huế, miền Trung lưu giữ những nhịp sống dịu dàng và giàu bản sắc.</p><a href="#booking">Tìm hiểu về kỳ nghỉ <span>↗</span></a></div>
      </section>
    </main>

    <footer id="footer" class="page-footer"><a class="brand" href="#top">LUMINA<span>®</span></a><span>Đón bạn bằng sự chăm sóc chân thành.</span><span>© 2026 Lumina Resorts</span></footer>
  </div>
</template>

<style scoped>
.lumina-page{--paper:#f5f2eb;--paper-deep:#eae5da;--ink:#282b24;--muted:#77786e;--line:#d6d2c7;--forest:#3a493b;--serif:'Playfair Display',Georgia,serif;--sans:'DM Sans',Arial,sans-serif;background:var(--paper);color:var(--ink);font-family:var(--sans);min-height:100vh;-webkit-font-smoothing:antialiased}.lumina-page *{box-sizing:border-box}.topbar{height:78px;padding:0 7.2%;display:flex;align-items:center;justify-content:space-between;background:var(--paper);position:relative;z-index:2}.brand{font:500 21px/1 var(--serif);letter-spacing:.22em;color:var(--ink);text-decoration:none}.brand span{font:9px var(--sans);vertical-align:top;letter-spacing:0;margin-left:3px}.top-nav{display:flex;align-items:center;gap:42px}.top-nav a,.account-link{font:500 10px var(--sans);letter-spacing:.13em;text-transform:uppercase;color:#5c5e54;text-decoration:none}.top-nav a:hover,.account-link:hover{color:#95734f}.account-link{border-bottom:1px solid #9e9d92;padding:9px 0}.account-link span{margin-left:10px}.signed-in{max-width:180px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}.hero{position:relative;height:min(700px,calc(100svh - 78px));min-height:560px;overflow:hidden;color:#fff;background:#3c4539}.hero-photo,.hero-overlay{position:absolute;inset:0}.hero-photo{background:url('https://images.unsplash.com/photo-1772004840079-e8f70d3b0879?auto=format&fit=crop&w=2400&q=85') center 52%/cover;transform:scale(1.01);animation:photo-settle 1.2s ease-out both}.hero-overlay{background:linear-gradient(90deg,rgba(17,25,19,.52),rgba(17,25,19,.08) 74%),linear-gradient(0deg,rgba(15,21,17,.3),transparent 40%)}.hero-content{position:absolute;top:48%;left:14%;transform:translateY(-45%);animation:copy-rise .8s .12s both}.kicker,.section-kicker{font-size:9px;font-weight:600;line-height:1.5;letter-spacing:.24em;color:#8b765d}.kicker{margin:0 0 26px;color:rgba(255,255,255,.83)}.hero h1,.booking-heading h2,.story-copy h2{font:400 clamp(47px,6vw,78px)/1.14 var(--serif);letter-spacing:-.035em;margin:0}.hero h1 em,.booking-heading h2 em,.story-copy h2 em{font-weight:400}.hero-description{margin:20px 0 28px;color:rgba(255,255,255,.84);font-size:12px;line-height:1.9;letter-spacing:.025em}.hero-link,.story-copy a{display:inline-flex;align-items:center;gap:18px;color:white;text-transform:uppercase;letter-spacing:.15em;font-size:9px;text-decoration:none;border-bottom:1px solid currentColor;padding-bottom:8px}.hero-link span{font-size:15px}.hero-meta{position:absolute;left:7.2%;right:7.2%;bottom:25px;display:flex;justify-content:space-between;font-size:8px;letter-spacing:.16em;color:rgba(255,255,255,.78)}.booking-section{max-width:1240px;margin:auto;padding:93px 42px 104px;display:grid;grid-template-columns:.7fr 1.55fr;gap:9%;align-items:start}.booking-heading .section-kicker{margin:0 0 19px}.booking-heading h2,.story-copy h2{font-size:clamp(35px,3.5vw,47px);line-height:1.25;letter-spacing:-.025em}.booking-heading>p:not(.section-kicker){max-width:290px;margin:16px 0 0;color:var(--muted);font-size:12px;line-height:1.8}.guest-mode{display:flex;align-items:center;gap:9px;margin-top:28px;color:#78796f;font-size:10px}.guest-mode strong{color:#42463d;font-weight:600}.mode-dot{width:6px;height:6px;background:#71816e;border-radius:50%}.booking-form{padding-top:7px}.form-section-label{display:flex;align-items:center;gap:13px;padding-bottom:13px;border-bottom:1px solid var(--line);color:#6f7167;font-size:9px;font-weight:600;letter-spacing:.18em}.form-section-label span{color:#a18b6d;font-family:var(--serif);font-size:14px;letter-spacing:0}.form-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));column-gap:30px;row-gap:19px;padding:22px 0 32px}.field{display:block;min-width:0}.field>span{display:block;margin-bottom:8px;color:#73756b;font-size:9px;letter-spacing:.12em;text-transform:uppercase}.field b{color:#a27451;font-weight:500}.field small{margin-left:5px;color:#a2a095;font-size:8px;letter-spacing:.06em;text-transform:none}.field input,.field select,.field textarea{display:block;width:100%;height:38px;padding:0 2px;border:0;border-bottom:1px solid var(--line);border-radius:0;outline:0;background:transparent;color:var(--ink);font:12px var(--sans);transition:border-color .2s}.field textarea{height:auto;min-height:51px;padding-top:10px;resize:vertical}.field input:focus,.field select:focus,.field textarea:focus{border-color:#71806e}.field input::placeholder,.field textarea::placeholder{color:#aaa99f}.field select{cursor:pointer}.full-field{grid-column:1/-1}.guest-label{margin-top:1px}.contact-fields{padding-bottom:24px}.form-bottom{display:flex;justify-content:space-between;align-items:center;gap:20px}.privacy-note{margin:0;color:#87877d;font-size:9px}.privacy-note span{margin-right:6px;color:#8a755d;font-size:15px}.submit-button{min-width:174px;min-height:47px;padding:0 19px;border:1px solid var(--forest);background:var(--forest);color:white;font:500 10px var(--sans);letter-spacing:.1em;cursor:pointer;transition:background .2s,color .2s}.submit-button span{margin-left:25px;font-size:15px}.submit-button:hover{background:transparent;color:var(--forest)}.feedback{margin:16px 0 0;padding:13px 15px;background:#e7ebe2;color:#465441;font-size:11px;line-height:1.7}.feedback.error{background:#f3e5df;color:#8d4835}.stay-story{display:grid;grid-template-columns:1.04fr .96fr;min-height:500px;background:var(--paper-deep)}.story-image{min-height:500px;background:url('https://images.unsplash.com/photo-1720456485611-a266e43e2bca?auto=format&fit=crop&w=1600&q=85') center/cover}.story-copy{display:flex;flex-direction:column;align-items:flex-start;justify-content:center;padding:75px clamp(45px,9vw,140px)}.story-copy .section-kicker{margin:0 0 18px}.story-copy>p:not(.section-kicker){max-width:345px;margin:20px 0 25px;color:#6f7168;font-size:12px;line-height:1.9}.story-copy a{color:var(--ink)}.page-footer{min-height:100px;padding:20px 7.2%;display:flex;align-items:center;justify-content:space-between;color:#77796f;font-size:9px;letter-spacing:.07em}.page-footer .brand{font-size:17px}@keyframes copy-rise{from{opacity:0;transform:translateY(calc(-45% + 18px))}to{opacity:1;transform:translateY(-45%)}}@keyframes photo-settle{from{transform:scale(1.05)}to{transform:scale(1.01)}}@media(max-width:900px){.topbar{padding:0 5%}.top-nav{gap:22px}.booking-section{grid-template-columns:1fr;gap:30px;padding:70px 8%}.booking-heading>p:not(.section-kicker){max-width:430px}.guest-mode{margin-top:18px}.story-copy{padding:55px 8%}}@media(max-width:600px){.topbar{height:68px}.brand{font-size:18px}.top-nav{display:none}.account-link{font-size:9px}.hero{height:630px;min-height:0}.hero-photo{background-position:57% center}.hero-overlay{background:linear-gradient(90deg,rgba(17,25,19,.46),rgba(17,25,19,.08)),linear-gradient(0deg,rgba(15,21,17,.34),transparent 58%)}.hero-content{left:8%;right:7%;top:46%}.hero h1{font-size:clamp(43px,12.5vw,60px)}.hero-description{font-size:11px}.hero-meta{left:8%;right:8%;bottom:20px;font-size:7px}.booking-section{padding:60px 7% 70px}.form-grid{grid-template-columns:1fr;row-gap:17px;padding:20px 0 27px}.full-field{grid-column:auto}.form-bottom{align-items:stretch;flex-direction:column}.submit-button{width:100%}.privacy-note{order:2}.stay-story{grid-template-columns:1fr}.story-image{min-height:320px}.story-copy{min-height:355px;padding:50px 8%}.page-footer{min-height:125px;padding:23px 7%;flex-wrap:wrap;gap:12px}.page-footer span:last-child{width:100%}}@media(prefers-reduced-motion:reduce){.lumina-page *{scroll-behavior:auto!important;animation-duration:.01ms!important;transition-duration:.01ms!important}}
.required-note{margin-top:10px!important;color:#8a755d!important;font-size:10px!important}.required-note b{color:#a36f52}.form-grid{column-gap:16px;row-gap:13px;padding-top:20px}.field{padding:12px 14px 10px;border:1px solid #e2ddd2;background:rgba(255,255,255,.44);transition:border-color .18s,background .18s}.field:focus-within{border-color:#83917c;background:rgba(255,255,255,.76)}.field>span{margin-bottom:6px;color:#56594f;font-size:10px;font-weight:600;letter-spacing:.07em;text-transform:none}.field b{margin-left:2px;color:#a66f52}.field small{margin:0}.field input,.field select,.field textarea{height:32px;font-size:13px}.field textarea{min-height:48px}.field-invalid,.field-invalid:focus-within{border-color:#b7826a;background:#fbf3ee}.field-invalid input,.field-invalid select{border-color:#c89a85}.field .field-error{display:block;margin-top:7px;color:#925e48;font-size:10px;font-weight:500;line-height:1.45;letter-spacing:0;text-transform:none}.field .field-hint{display:block;margin-top:6px;color:#697661;font-size:10px;letter-spacing:0;text-transform:none}.feedback.error{border-left:2px solid #a66f52;background:#f4eae3;color:#80523f}
.room-picker-wrap{padding:16px;border:1px solid #e2ddd2;background:rgba(255,255,255,.42);transition:border-color .18s,background .18s}.room-picker-invalid{border-color:#b7826a;background:#fbf3ee}.room-picker-heading{display:flex;align-items:flex-start;justify-content:space-between;gap:14px;margin-bottom:13px}.room-picker-title{color:#45493f;font-size:12px;font-weight:600}.room-picker-title b{color:#a66f52}.room-picker-heading p{margin:5px 0 0;color:#77796f;font-size:10px;line-height:1.5}.room-picker-count{padding:5px 7px;border:1px solid #d9d4c8;color:#77796f;font-size:8px;letter-spacing:.08em;white-space:nowrap}.room-card-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:10px}.room-card{position:relative;display:flex;align-items:stretch;gap:12px;min-width:0;min-height:108px;padding:7px;border:1px solid #e2ddd2;background:#f9f7f2;color:var(--ink);text-align:left;cursor:pointer;transition:border-color .18s,background .18s,transform .18s}.room-card:hover{border-color:#aab19f;transform:translateY(-1px)}.room-card.selected{border-color:#596a57;background:#eff1e9;box-shadow:inset 0 0 0 1px #596a57}.room-card-image{width:108px;min-width:108px;min-height:92px;object-fit:cover;background:#e8e3d8}.room-card-copy{display:flex;flex-direction:column;align-items:flex-start;justify-content:center;gap:5px;min-width:0;padding:5px 16px 5px 0}.room-card-name{color:#363b33;font:500 14px/1.3 var(--serif)}.room-card-detail{color:#66695f;font-size:10px}.room-card-status{color:#8b765d;font-size:9px;line-height:1.4}.room-card-check{position:absolute;top:9px;right:9px;display:grid;width:17px;height:17px;place-items:center;border:1px solid #d0ccbf;border-radius:50%;color:transparent;font-size:10px}.room-card.selected .room-card-check{border-color:var(--forest);background:var(--forest);color:#fff}.room-picker-error{display:block;margin-top:10px;color:#925e48;font-size:10px}.selected-room-preview{display:grid;grid-template-columns:180px 1fr;gap:17px;align-items:center;margin-top:14px;padding:12px;background:#eeece3}.selected-room-preview img{width:180px;height:112px;object-fit:cover;background:#e2ddd2}.selected-room-preview>div{display:flex;flex-direction:column;align-items:flex-start;gap:6px}.preview-kicker{color:#8b765d;font-size:8px;font-weight:600;letter-spacing:.15em}.selected-room-preview strong{color:#33382f;font:500 18px/1.3 var(--serif)}.selected-room-preview>div>span:not(.preview-kicker){color:#65685e;font-size:11px}.selected-room-preview small{color:#87877d;font-size:9px}.availability-note{display:flex;align-items:flex-start;gap:8px;margin:12px 0 0;color:#707269;font-size:10px;line-height:1.6}.availability-note span{color:#8c795f}.form-bottom{align-items:flex-start}.privacy-consent{position:relative;display:flex;flex:1;align-items:flex-start;gap:10px;max-width:410px;padding:5px 0;color:#5e6258;font-size:10px;line-height:1.65;cursor:pointer}.privacy-consent input{position:absolute;width:18px;height:18px;margin:0;opacity:0;cursor:pointer}.consent-check{display:grid;width:18px;height:18px;min-width:18px;place-items:center;border:1px solid #a8a89a;background:#fbfaf6;color:transparent;font-size:11px;transition:background .18s,border-color .18s,color .18s}.privacy-consent input:checked+.consent-check{border-color:var(--forest);background:var(--forest);color:#fff}.privacy-consent input:focus-visible+.consent-check{outline:2px solid #8c9a83;outline-offset:2px}.consent-copy b{color:#a66f52}.consent-invalid .consent-check{border-color:#b7826a;background:#fbf3ee}.consent-error{display:block;margin-top:8px;color:#925e48;font-size:10px}.submit-button{flex:0 0 auto}.field>span small{display:inline;color:#88887d;font-weight:400;font-size:9px}
@media(max-width:600px){.room-picker-wrap{padding:12px}.room-picker-heading{flex-direction:column;gap:8px}.room-card-grid{gap:8px}.room-card{flex-direction:column;gap:8px;min-height:0;padding:6px}.room-card-image{width:100%;height:88px;min-width:0;min-height:0}.room-card-copy{gap:4px;padding:2px 2px 4px}.room-card-name{font-size:13px}.room-card-detail,.room-card-status{font-size:9px}.room-card-check{top:10px;right:10px;background:#f9f7f2}.selected-room-preview{grid-template-columns:1fr;gap:10px}.selected-room-preview img{width:100%;height:150px}.form-bottom{gap:12px}.privacy-consent{max-width:none}.submit-button{width:100%}}
.lumina-page{--serif:'Lora',Georgia,serif;--sans:'Inter',Arial,sans-serif}.hero h1{line-height:1.13;letter-spacing:-.028em}.hero-description{font-size:14px;line-height:1.8}.main-nav a,.account-link{font-size:11px}.booking-heading>p:not(.section-kicker){font-size:13px}.required-note{font-size:11px!important}.field>span{font-size:11px}.field input,.field select,.field textarea{height:36px;font-size:14px}.field textarea{min-height:56px}.field .field-error{font-size:11px}.field .field-hint{font-size:11px}.room-picker-heading p{font-size:11px}.room-picker-title{font-size:13px}.room-card-copy{overflow-wrap:anywhere}.room-card-name{font-size:15px}.room-card-detail{font-size:11px}.room-card-status{font-size:10px}.selected-room-preview>div{min-width:0}.selected-room-preview strong{font-size:19px;overflow-wrap:anywhere}.selected-room-preview>div>span:not(.preview-kicker){font-size:12px}.selected-room-preview small{font-size:10px;line-height:1.5}.availability-note,.privacy-consent{font-size:11px;line-height:1.65}.consent-error,.room-picker-error{font-size:11px}.submit-button{font-size:11px}.feedback{font-size:12px;overflow-wrap:anywhere}.field input,.field select,.field textarea{min-width:0;max-width:100%}
@media(max-width:600px){.hero-description{font-size:13px}.field>span{font-size:11px}.field input,.field select,.field textarea{font-size:14px}.room-card-name{font-size:14px}.room-card-detail,.room-card-status{font-size:10px}.room-picker-count{font-size:8px}.selected-room-preview strong{font-size:18px}}
.hero-photo{filter:saturate(.78) sepia(.1)}
.hero-annotation{position:absolute;top:24%;right:10%;display:flex;align-items:center;gap:13px;padding:13px 17px;border:1px solid rgba(255,255,255,.38);background:rgba(35,40,32,.18);backdrop-filter:blur(8px);animation:annotation-float 5s ease-in-out infinite;color:#fff}
.annotation-spark{font-size:24px;color:#e7d4b3;animation:annotation-turn 18s linear infinite}
.hero-annotation>span:nth-child(2){display:flex;flex-direction:column;gap:5px}
.hero-annotation small{font-size:8px;letter-spacing:.16em;color:rgba(255,255,255,.72)}
.hero-annotation strong{font:400 15px/1.3 var(--serif);letter-spacing:.02em}
.annotation-index{align-self:flex-end;margin-left:10px;padding-left:12px;border-left:1px solid rgba(255,255,255,.35);font-size:8px;letter-spacing:.12em;color:rgba(255,255,255,.75)}
.hero-side-note{position:absolute;right:3%;top:43%;writing-mode:vertical-rl;font-size:8px;letter-spacing:.22em;color:rgba(255,255,255,.72);animation:annotation-float 6s ease-in-out infinite reverse}
@keyframes annotation-float{0%,100%{transform:translateY(0)}50%{transform:translateY(-8px)}}
@keyframes annotation-turn{to{transform:rotate(360deg)}}
@media(max-width:900px){.hero-annotation{right:7%;top:13%;transform:scale(.94);transform-origin:top right}.hero-side-note{right:2%}}
@media(max-width:600px){.hero-annotation{top:8%;right:7%;gap:9px;padding:10px 12px;animation-duration:6s}.hero-annotation strong{font-size:13px}.hero-annotation small{font-size:7px}.annotation-index{font-size:7px;margin-left:2px;padding-left:8px}.hero-side-note{display:none}}
@media(prefers-reduced-motion:reduce){.hero-annotation,.annotation-spark,.hero-side-note{animation:none}}
</style>
