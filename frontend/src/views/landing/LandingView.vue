<script setup lang="ts">
import { computed, onMounted, reactive, ref } from 'vue';
import { useAuthStore } from '@/stores/auth.store';

const auth = useAuthStore();
const roomTypes = [
  { value: 'garden', label: 'Garden Room', detail: 'Vườn riêng · 42 m²' },
  { value: 'ocean', label: 'Ocean View Room', detail: 'Hướng biển · 48 m²' },
  { value: 'suite', label: 'Lumina Suite', detail: 'Phòng khách riêng · 72 m²' },
  { value: 'villa', label: 'Private Pool Villa', detail: 'Hồ bơi riêng · 110 m²' },
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
  roomType: 'ocean',
  note: '',
});

const feedback = ref('');
const feedbackIsError = ref(false);
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

function submitBooking() {
  feedback.value = '';
  feedbackIsError.value = false;
  if (nights.value < 1) {
    feedbackIsError.value = true;
    feedback.value = 'Ngày trả phòng cần sau ngày nhận phòng ít nhất một đêm.';
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
        <div class="hero-photo" role="img" aria-label="Khu nghỉ dưỡng giữa núi rừng và biển xanh" />
        <div class="hero-overlay" />
        <div class="hero-content">
          <p class="kicker">NHA TRANG · VIỆT NAM</p>
          <h1 id="hero-title">Trở về với<br /><em>những điều giản dị.</em></h1>
          <p class="hero-description">Một chốn nghỉ yên bình để sống chậm,<br class="desktop-only" /> hít thở sâu và cảm nhận nhiều hơn.</p>
          <a class="hero-link" href="#booking">Khám phá kỳ nghỉ <span>↓</span></a>
        </div>
        <div class="hero-meta"><span>12° 14' N — 109° 11' E</span><span>VỊNH BIỂN MIỀN TRUNG</span></div>
      </section>

      <section id="booking" class="booking-section">
        <div class="booking-heading">
          <p class="section-kicker">KỲ NGHỈ CỦA BẠN</p>
          <h2>Hãy để chúng tôi<br /><em>lo phần còn lại.</em></h2>
          <p>Chia sẻ một vài thông tin để chúng tôi chuẩn bị đón tiếp bạn chu đáo.</p>
          <div class="guest-mode">
            <span class="mode-dot" />
            <span v-if="auth.isAuthenticated()">Đang đặt với hồ sơ <strong>{{ auth.user?.fullName }}</strong></span>
            <span v-else>Đặt phòng với tư cách <strong>khách</strong></span>
          </div>
        </div>

        <form class="booking-form" @submit.prevent="submitBooking">
          <div class="form-section-label"><span>01</span> THÔNG TIN LƯU TRÚ</div>
          <div class="form-grid stay-fields">
            <label class="field" for="check-in"><span>Ngày nhận phòng <b>*</b></span><input id="check-in" v-model="form.checkIn" type="date" :min="isoDate(today)" required @change="updateCheckoutMinimum" /></label>
            <label class="field" for="check-out"><span>Ngày trả phòng <b>*</b></span><input id="check-out" v-model="form.checkOut" type="date" :min="checkoutMin" required /></label>
            <label class="field" for="arrival-time"><span>Giờ nhận phòng dự kiến</span><select id="arrival-time" v-model="form.checkInTime"><option>14:00</option><option>15:00</option><option>16:00</option><option>17:00</option><option>18:00</option><option>19:00</option><option>20:00</option></select></label>
            <label class="field" for="departure-time"><span>Giờ trả phòng dự kiến</span><select id="departure-time" v-model="form.checkOutTime"><option>08:00</option><option>09:00</option><option>10:00</option><option>11:00</option><option>12:00</option><option>13:00</option></select></label>
            <label class="field" for="guests"><span>Số khách</span><select id="guests" v-model.number="form.guests"><option :value="1">1 khách</option><option :value="2">2 khách</option><option :value="3">3 khách</option><option :value="4">4 khách</option><option :value="5">5 khách</option><option :value="6">6 khách</option></select></label>
            <label class="field" for="room-type"><span>Hạng phòng <b>*</b></span><select id="room-type" v-model="form.roomType"><option v-for="room in roomTypes" :key="room.value" :value="room.value">{{ room.label }}</option></select></label>
          </div>

          <div class="form-section-label guest-label"><span>02</span> THÔNG TIN KHÁCH ĐẠI DIỆN</div>
          <div class="form-grid contact-fields">
            <label class="field" for="full-name"><span>Họ và tên <b>*</b></span><input id="full-name" v-model.trim="form.fullName" type="text" autocomplete="name" placeholder="Tên trên giấy tờ tuỳ thân" required minlength="2" /></label>
            <label class="field" for="phone"><span>Số điện thoại <b>*</b></span><input id="phone" v-model.trim="form.phone" type="tel" autocomplete="tel" placeholder="090 123 4567" required pattern="[0-9+ ()-]{9,16}" title="Vui lòng nhập số điện thoại hợp lệ" /></label>
            <label class="field" for="identity"><span>Số CCCD / hộ chiếu <b>*</b></span><input id="identity" v-model.trim="form.identityNumber" type="text" placeholder="Số trên giấy tờ tuỳ thân" required minlength="9" maxlength="20" /></label>
            <label class="field" for="email"><span>Email <small>Không bắt buộc</small></span><input id="email" v-model.trim="form.email" type="email" autocomplete="email" placeholder="ban@email.com" /></label>
            <label class="field full-field" for="note"><span>Ghi chú cho kỳ nghỉ <small>Không bắt buộc</small></span><textarea id="note" v-model.trim="form.note" rows="2" placeholder="Giờ đến dự kiến, yêu cầu đặc biệt..."></textarea></label>
          </div>

          <div class="form-bottom">
            <p class="privacy-note"><span>◇</span> Thông tin của bạn được giữ riêng tư và bảo mật.</p>
            <button type="submit" class="submit-button">Gửi yêu cầu <span>→</span></button>
          </div>
          <p v-if="feedback" class="feedback" :class="{ error: feedbackIsError }" role="status" aria-live="polite">{{ feedback }}</p>
        </form>
      </section>

      <section id="stay" class="stay-story">
        <div class="story-image" role="img" aria-label="Không gian nghỉ dưỡng thanh tĩnh giữa thiên nhiên" />
        <div class="story-copy"><p class="section-kicker">MỘT NƠI ĐỂ THUỘC VỀ</p><h2>Vẻ đẹp của<br /><em>những khoảng lặng.</em></h2><p>Ẩn mình bên bờ biển miền Trung, Lumina mở ra một nhịp sống khác — dịu dàng, riêng tư và gần gũi với thiên nhiên.</p><a href="#booking">Tìm hiểu về kỳ nghỉ <span>↗</span></a></div>
      </section>
    </main>

    <footer id="footer" class="page-footer"><a class="brand" href="#top">LUMINA<span>®</span></a><span>Đón bạn bằng sự chăm sóc chân thành.</span><span>© 2026 Lumina Resorts</span></footer>
  </div>
</template>

<style scoped>
.lumina-page{--paper:#f5f2eb;--paper-deep:#eae5da;--ink:#282b24;--muted:#77786e;--line:#d6d2c7;--forest:#3a493b;--serif:'Playfair Display',Georgia,serif;--sans:'DM Sans',Arial,sans-serif;background:var(--paper);color:var(--ink);font-family:var(--sans);min-height:100vh;-webkit-font-smoothing:antialiased}.lumina-page *{box-sizing:border-box}.topbar{height:78px;padding:0 7.2%;display:flex;align-items:center;justify-content:space-between;background:var(--paper);position:relative;z-index:2}.brand{font:500 21px/1 var(--serif);letter-spacing:.22em;color:var(--ink);text-decoration:none}.brand span{font:9px var(--sans);vertical-align:top;letter-spacing:0;margin-left:3px}.top-nav{display:flex;align-items:center;gap:42px}.top-nav a,.account-link{font:500 10px var(--sans);letter-spacing:.13em;text-transform:uppercase;color:#5c5e54;text-decoration:none}.top-nav a:hover,.account-link:hover{color:#95734f}.account-link{border-bottom:1px solid #9e9d92;padding:9px 0}.account-link span{margin-left:10px}.signed-in{max-width:180px;overflow:hidden;text-overflow:ellipsis;white-space:nowrap}.hero{position:relative;height:min(700px,calc(100svh - 78px));min-height:560px;overflow:hidden;color:#fff;background:#3c4539}.hero-photo,.hero-overlay{position:absolute;inset:0}.hero-photo{background:url('https://images.unsplash.com/photo-1530789253388-582c481c54b0?auto=format&fit=crop&w=2400&q=85') center 52%/cover;transform:scale(1.01);animation:photo-settle 1.2s ease-out both}.hero-overlay{background:linear-gradient(90deg,rgba(17,25,19,.52),rgba(17,25,19,.08) 74%),linear-gradient(0deg,rgba(15,21,17,.3),transparent 40%)}.hero-content{position:absolute;top:48%;left:14%;transform:translateY(-45%);animation:copy-rise .8s .12s both}.kicker,.section-kicker{font-size:9px;font-weight:600;line-height:1.5;letter-spacing:.24em;color:#8b765d}.kicker{margin:0 0 26px;color:rgba(255,255,255,.83)}.hero h1,.booking-heading h2,.story-copy h2{font:400 clamp(47px,6vw,78px)/1.14 var(--serif);letter-spacing:-.035em;margin:0}.hero h1 em,.booking-heading h2 em,.story-copy h2 em{font-weight:400}.hero-description{margin:20px 0 28px;color:rgba(255,255,255,.84);font-size:12px;line-height:1.9;letter-spacing:.025em}.hero-link,.story-copy a{display:inline-flex;align-items:center;gap:18px;color:white;text-transform:uppercase;letter-spacing:.15em;font-size:9px;text-decoration:none;border-bottom:1px solid currentColor;padding-bottom:8px}.hero-link span{font-size:15px}.hero-meta{position:absolute;left:7.2%;right:7.2%;bottom:25px;display:flex;justify-content:space-between;font-size:8px;letter-spacing:.16em;color:rgba(255,255,255,.78)}.booking-section{max-width:1240px;margin:auto;padding:93px 42px 104px;display:grid;grid-template-columns:.7fr 1.55fr;gap:9%;align-items:start}.booking-heading .section-kicker{margin:0 0 19px}.booking-heading h2,.story-copy h2{font-size:clamp(35px,3.5vw,47px);line-height:1.25;letter-spacing:-.025em}.booking-heading>p:not(.section-kicker){max-width:290px;margin:16px 0 0;color:var(--muted);font-size:12px;line-height:1.8}.guest-mode{display:flex;align-items:center;gap:9px;margin-top:28px;color:#78796f;font-size:10px}.guest-mode strong{color:#42463d;font-weight:600}.mode-dot{width:6px;height:6px;background:#71816e;border-radius:50%}.booking-form{padding-top:7px}.form-section-label{display:flex;align-items:center;gap:13px;padding-bottom:13px;border-bottom:1px solid var(--line);color:#6f7167;font-size:9px;font-weight:600;letter-spacing:.18em}.form-section-label span{color:#a18b6d;font-family:var(--serif);font-size:14px;letter-spacing:0}.form-grid{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));column-gap:30px;row-gap:19px;padding:22px 0 32px}.field{display:block;min-width:0}.field>span{display:block;margin-bottom:8px;color:#73756b;font-size:9px;letter-spacing:.12em;text-transform:uppercase}.field b{color:#a27451;font-weight:500}.field small{margin-left:5px;color:#a2a095;font-size:8px;letter-spacing:.06em;text-transform:none}.field input,.field select,.field textarea{display:block;width:100%;height:38px;padding:0 2px;border:0;border-bottom:1px solid var(--line);border-radius:0;outline:0;background:transparent;color:var(--ink);font:12px var(--sans);transition:border-color .2s}.field textarea{height:auto;min-height:51px;padding-top:10px;resize:vertical}.field input:focus,.field select:focus,.field textarea:focus{border-color:#71806e}.field input::placeholder,.field textarea::placeholder{color:#aaa99f}.field select{cursor:pointer}.full-field{grid-column:1/-1}.guest-label{margin-top:1px}.contact-fields{padding-bottom:24px}.form-bottom{display:flex;justify-content:space-between;align-items:center;gap:20px}.privacy-note{margin:0;color:#87877d;font-size:9px}.privacy-note span{margin-right:6px;color:#8a755d;font-size:15px}.submit-button{min-width:174px;min-height:47px;padding:0 19px;border:1px solid var(--forest);background:var(--forest);color:white;font:500 10px var(--sans);letter-spacing:.1em;cursor:pointer;transition:background .2s,color .2s}.submit-button span{margin-left:25px;font-size:15px}.submit-button:hover{background:transparent;color:var(--forest)}.feedback{margin:16px 0 0;padding:13px 15px;background:#e7ebe2;color:#465441;font-size:11px;line-height:1.7}.feedback.error{background:#f3e5df;color:#8d4835}.stay-story{display:grid;grid-template-columns:1.04fr .96fr;min-height:500px;background:var(--paper-deep)}.story-image{min-height:500px;background:url('https://images.unsplash.com/photo-1571896349842-33c89424de2d?auto=format&fit=crop&w=1600&q=85') center/cover}.story-copy{display:flex;flex-direction:column;align-items:flex-start;justify-content:center;padding:75px clamp(45px,9vw,140px)}.story-copy .section-kicker{margin:0 0 18px}.story-copy>p:not(.section-kicker){max-width:345px;margin:20px 0 25px;color:#6f7168;font-size:12px;line-height:1.9}.story-copy a{color:var(--ink)}.page-footer{min-height:100px;padding:20px 7.2%;display:flex;align-items:center;justify-content:space-between;color:#77796f;font-size:9px;letter-spacing:.07em}.page-footer .brand{font-size:17px}@keyframes copy-rise{from{opacity:0;transform:translateY(calc(-45% + 18px))}to{opacity:1;transform:translateY(-45%)}}@keyframes photo-settle{from{transform:scale(1.05)}to{transform:scale(1.01)}}@media(max-width:900px){.topbar{padding:0 5%}.top-nav{gap:22px}.booking-section{grid-template-columns:1fr;gap:30px;padding:70px 8%}.booking-heading>p:not(.section-kicker){max-width:430px}.guest-mode{margin-top:18px}.story-copy{padding:55px 8%}}@media(max-width:600px){.topbar{height:68px}.brand{font-size:18px}.top-nav{display:none}.account-link{font-size:9px}.hero{height:630px;min-height:0}.hero-photo{background-position:57% center}.hero-overlay{background:linear-gradient(90deg,rgba(17,25,19,.46),rgba(17,25,19,.08)),linear-gradient(0deg,rgba(15,21,17,.34),transparent 58%)}.hero-content{left:8%;right:7%;top:46%}.hero h1{font-size:clamp(43px,12.5vw,60px)}.hero-description{font-size:11px}.hero-meta{left:8%;right:8%;bottom:20px;font-size:7px}.booking-section{padding:60px 7% 70px}.form-grid{grid-template-columns:1fr;row-gap:17px;padding:20px 0 27px}.full-field{grid-column:auto}.form-bottom{align-items:stretch;flex-direction:column}.submit-button{width:100%}.privacy-note{order:2}.stay-story{grid-template-columns:1fr}.story-image{min-height:320px}.story-copy{min-height:355px;padding:50px 8%}.page-footer{min-height:125px;padding:23px 7%;flex-wrap:wrap;gap:12px}.page-footer span:last-child{width:100%}}@media(prefers-reduced-motion:reduce){.lumina-page *{scroll-behavior:auto!important;animation-duration:.01ms!important;transition-duration:.01ms!important}}
</style>
