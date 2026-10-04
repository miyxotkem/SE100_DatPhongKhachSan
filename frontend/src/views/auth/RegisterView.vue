<script setup lang="ts">
import { computed, reactive, ref } from 'vue';
import { useRouter } from 'vue-router';
import apiClient from '@/services/api.client';
import { useAuthStore } from '@/stores/auth.store';

const router = useRouter();
const auth = useAuthStore();
const form = reactive({ fullName: '', phone: '', email: '', password: '', confirmPassword: '' });
const showPassword = ref(false);
const showConfirmPassword = ref(false);
const attempted = ref(false);
const submitting = ref(false);
const formError = ref('');

const errors = computed(() => ({
  fullName: form.fullName.trim().length < 2 ? 'Vui lòng nhập họ tên từ 2 ký tự.' : '',
  phone: !/^[0-9+ ()-]{9,16}$/.test(form.phone.trim()) ? 'Vui lòng nhập số điện thoại hợp lệ.' : '',
  email: !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email.trim()) ? 'Email chưa đúng định dạng.' : '',
  password: form.password.length < 6 ? 'Mật khẩu cần có ít nhất 6 ký tự.' : '',
  confirmPassword: !form.confirmPassword
    ? 'Vui lòng nhập lại mật khẩu.'
    : form.confirmPassword !== form.password
      ? 'Hai mật khẩu chưa trùng khớp.'
      : '',
}));

interface RegisterResponse {
  accessToken: string;
  user: {
    id: string;
    email: string;
    full_name: string;
    phone_number?: string;
    avatar_url?: string;
  };
}

async function submitRegistration() {
  attempted.value = true;
  formError.value = '';
  if (Object.values(errors.value).some(Boolean)) return;

  submitting.value = true;
  try {
    const result = await apiClient.post('/auth/register', {
      email: form.email.trim(),
      password: form.password,
      full_name: form.fullName.trim(),
      phone_number: form.phone.trim(),
    }) as unknown as RegisterResponse;

    auth.setAuth(result.accessToken, {
      id: result.user.id,
      email: result.user.email,
      fullName: result.user.full_name,
      phoneNumber: result.user.phone_number,
      avatarUrl: result.user.avatar_url,
    });
    await router.push('/dashboard');
  } catch (error) {
    const response = error as { message?: string | string[] };
    formError.value = Array.isArray(response.message)
      ? response.message[0]
      : response.message || 'Chưa thể tạo tài khoản lúc này. Vui lòng thử lại sau.';
  } finally {
    submitting.value = false;
  }
}
</script>

<template>
  <main class="register-page">
    <header class="register-header">
      <RouterLink class="register-brand" to="/" aria-label="Lumina, về trang chủ">LUMINA<span>®</span></RouterLink>
      <RouterLink class="return-link" to="/">Về trang chủ <span>↗</span></RouterLink>
    </header>

    <section class="register-layout">
      <aside class="register-intro">
        <p class="register-kicker">LUMINA · NHA TRANG</p>
        <h1>Một kỳ nghỉ<br />bắt đầu từ<br /><em>một lời chào.</em></h1>
        <p class="intro-note">Tạo tài khoản để lưu lại thông tin của bạn và chuẩn bị cho những lần ghé thăm tiếp theo.</p>
        <div class="intro-detail"><span class="detail-rule" /> <span>RIÊNG TƯ · CHẬM RÃI · CHÂN THÀNH</span></div>
      </aside>

      <section class="register-panel" aria-labelledby="register-title">
        <div class="panel-heading">
          <p class="panel-kicker">TÀI KHOẢN LUMINA</p>
          <h2 id="register-title">Đăng ký</h2>
          <p>Các mục có dấu <b>*</b> là bắt buộc.</p>
        </div>

        <form novalidate @submit.prevent="submitRegistration">
          <div class="register-fields">
            <label class="register-field" :class="{ invalid: attempted && errors.fullName }" for="register-name">
              <span>Họ và tên <b>*</b></span>
              <input id="register-name" v-model.trim="form.fullName" type="text" autocomplete="name" placeholder="Tên trên giấy tờ tuỳ thân" aria-describedby="name-error" />
              <small v-if="attempted && errors.fullName" id="name-error" class="input-error">{{ errors.fullName }}</small>
            </label>

            <label class="register-field" :class="{ invalid: attempted && errors.phone }" for="register-phone">
              <span>Số điện thoại <b>*</b></span>
              <input id="register-phone" v-model.trim="form.phone" type="tel" autocomplete="tel" placeholder="090 123 4567" aria-describedby="phone-error" />
              <small v-if="attempted && errors.phone" id="phone-error" class="input-error">{{ errors.phone }}</small>
            </label>

            <label class="register-field" :class="{ invalid: attempted && errors.email }" for="register-email">
              <span>Email <b>*</b></span>
              <input id="register-email" v-model.trim="form.email" type="email" autocomplete="email" placeholder="ban@email.com" aria-describedby="email-error" />
              <small v-if="attempted && errors.email" id="email-error" class="input-error">{{ errors.email }}</small>
            </label>

            <label class="register-field" :class="{ invalid: attempted && errors.password }" for="register-password">
              <span>Mật khẩu <b>*</b></span>
              <span class="password-control"><input id="register-password" v-model="form.password" :type="showPassword ? 'text' : 'password'" autocomplete="new-password" placeholder="Ít nhất 6 ký tự" aria-describedby="password-hint password-error" /><button type="button" class="visibility-toggle" :aria-label="showPassword ? 'Ẩn mật khẩu' : 'Hiện mật khẩu'" @click="showPassword = !showPassword">{{ showPassword ? 'Ẩn' : 'Hiện' }}</button></span>
              <small id="password-hint" class="field-hint">Dùng ít nhất 6 ký tự.</small>
              <small v-if="attempted && errors.password" id="password-error" class="input-error">{{ errors.password }}</small>
            </label>

            <label class="register-field" :class="{ invalid: attempted && errors.confirmPassword }" for="register-confirm-password">
              <span>Nhập lại mật khẩu <b>*</b></span>
              <span class="password-control"><input id="register-confirm-password" v-model="form.confirmPassword" :type="showConfirmPassword ? 'text' : 'password'" autocomplete="new-password" placeholder="Nhập lại mật khẩu của bạn" aria-describedby="confirm-password-error" /><button type="button" class="visibility-toggle" :aria-label="showConfirmPassword ? 'Ẩn mật khẩu' : 'Hiện mật khẩu'" @click="showConfirmPassword = !showConfirmPassword">{{ showConfirmPassword ? 'Ẩn' : 'Hiện' }}</button></span>
              <small v-if="attempted && errors.confirmPassword" id="confirm-password-error" class="input-error">{{ errors.confirmPassword }}</small>
            </label>
          </div>

          <p v-if="formError" class="form-error" role="alert">{{ formError }}</p>
          <button class="register-submit" type="submit" :disabled="submitting">
            {{ submitting ? 'Đang tạo tài khoản…' : 'Tạo tài khoản' }} <span aria-hidden="true">→</span>
          </button>
          <p class="login-prompt">Bạn đã có tài khoản? <RouterLink to="/login">Đăng nhập</RouterLink></p>
        </form>
      </section>
    </section>
    <footer class="register-footer"><span>© 2026 Lumina Resorts</span><span>Đón bạn bằng sự chăm sóc chân thành.</span></footer>
  </main>
</template>

<style scoped>
.register-page{--paper:#f5f2eb;--paper-deep:#eae5da;--ink:#282b24;--muted:#77786e;--line:#d8d3c8;--forest:#3a493b;--serif:'Playfair Display',Georgia,serif;--sans:'DM Sans',Arial,sans-serif;min-height:100vh;background:var(--paper);color:var(--ink);font-family:var(--sans);padding:0 7.2%;-webkit-font-smoothing:antialiased}.register-page *{box-sizing:border-box}.register-header{height:78px;display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid #e5e0d6}.register-brand{font:500 21px/1 var(--serif);letter-spacing:.22em;color:var(--ink);text-decoration:none}.register-brand span{font:9px var(--sans);vertical-align:top;letter-spacing:0;margin-left:3px}.return-link{font-size:10px;letter-spacing:.1em;text-transform:uppercase;color:#65675d;text-decoration:none}.return-link span{margin-left:10px}.register-layout{width:min(1080px,100%);min-height:calc(100svh - 145px);margin:0 auto;padding:65px 0 60px;display:grid;grid-template-columns:.9fr 1.1fr;gap:10%;align-items:center}.register-intro{padding:15px 0 25px}.register-kicker,.panel-kicker{margin:0 0 24px;color:#89775f;font-size:9px;font-weight:600;letter-spacing:.24em}.register-intro h1{font:400 clamp(45px,5.5vw,70px)/1.16 var(--serif);letter-spacing:-.035em;margin:0 0 23px}.register-intro h1 em{font-weight:400}.intro-note{max-width:340px;margin:0;color:#71736a;font-size:12px;line-height:1.9}.intro-detail{display:flex;align-items:center;gap:12px;margin-top:55px;color:#858277;font-size:8px;letter-spacing:.17em}.detail-rule{width:35px;height:1px;background:#a58e70}.register-panel{padding:38px 40px 32px;background:rgba(255,255,255,.54);border:1px solid #e6e1d7;box-shadow:0 16px 45px rgba(62,57,44,.045)}.panel-heading{margin-bottom:25px}.panel-kicker{margin:0 0 10px;font-size:8px}.panel-heading h2{margin:0 0 9px;font:400 36px/1.2 var(--serif)}.panel-heading>p:last-child{margin:0;color:#818176;font-size:10px}.panel-heading b{color:#a66f52}.register-fields{display:grid;grid-template-columns:1fr 1fr;gap:15px 14px}.register-field{display:block;min-width:0;padding:12px 13px 10px;border:1px solid #e2ddd2;background:rgba(255,255,255,.62);transition:border-color .18s,background .18s}.register-field:focus-within{border-color:#83917c;background:#fff}.register-field>span:first-child{display:block;margin-bottom:7px;color:#56594f;font-size:10px;font-weight:600;letter-spacing:.06em}.register-field>span b{margin-left:2px;color:#a66f52}.register-field>input,.password-control input{display:block;width:100%;height:31px;padding:0 1px;border:0;border-bottom:1px solid #d9d4c9;border-radius:0;outline:0;background:transparent;color:var(--ink);font:12px var(--sans)}.register-field input::placeholder{color:#aaa99f}.register-field input:focus{border-color:#71806e}.register-field:nth-child(3),.register-field:nth-child(4),.register-field:nth-child(5){grid-column:1/-1}.password-control{display:flex;align-items:center;gap:10px}.password-control input{flex:1;min-width:0}.visibility-toggle{padding:4px 0;border:0;background:transparent;color:#78816f;font:500 9px var(--sans);cursor:pointer}.visibility-toggle:hover{color:#3a493b}.field-hint,.input-error{display:block;margin-top:7px;font-size:9px;line-height:1.45;letter-spacing:0}.field-hint{color:#858579}.input-error{color:#925e48}.register-field.invalid{border-color:#b7826a;background:#fbf3ee}.register-field.invalid input{border-color:#c89a85}.form-error{margin:16px 0 0;padding:11px 13px;border-left:2px solid #a66f52;background:#f4eae3;color:#80523f;font-size:10px;line-height:1.6}.register-submit{width:100%;min-height:47px;margin-top:20px;padding:0 16px;border:1px solid var(--forest);background:var(--forest);color:white;font:500 10px var(--sans);letter-spacing:.12em;text-transform:uppercase;cursor:pointer;transition:background .2s,color .2s}.register-submit span{float:right;font-size:15px}.register-submit:hover:not(:disabled){background:transparent;color:var(--forest)}.register-submit:disabled{opacity:.65;cursor:wait}.login-prompt{margin:18px 0 0;text-align:center;color:#77796f;font-size:10px}.login-prompt a{color:#465442;text-decoration:underline;text-underline-offset:3px}.register-footer{min-height:67px;display:flex;align-items:center;justify-content:space-between;border-top:1px solid #e5e0d6;color:#858277;font-size:9px;letter-spacing:.07em}@media(max-width:800px){.register-page{padding:0 5%}.register-layout{grid-template-columns:1fr;gap:25px;max-width:520px;padding:42px 0}.register-intro{padding:0}.register-intro h1{font-size:51px}.intro-detail{margin-top:22px}.register-panel{padding:30px 27px}}@media(max-width:520px){.register-page{padding:0 6%}.register-header{height:68px}.register-brand{font-size:18px}.register-layout{padding:34px 0}.register-intro h1{font-size:43px}.register-panel{padding:25px 18px}.register-fields{grid-template-columns:1fr;gap:12px}.register-field:nth-child(n){grid-column:auto}.register-footer{align-items:flex-start;flex-direction:column;justify-content:center;gap:6px;padding:15px 0}}
.register-page{--serif:'Lora',Georgia,serif;--sans:'Inter',Arial,sans-serif}.register-layout{grid-template-columns:minmax(0,.9fr) minmax(0,1.1fr);gap:clamp(32px,7vw,96px)}.register-panel{min-width:0}.register-intro h1{line-height:1.12;letter-spacing:-.028em}.intro-note{font-size:13px}.panel-heading>p:last-child{font-size:11px}.register-fields{grid-template-columns:repeat(2,minmax(0,1fr))}.register-field{min-width:0}.register-field>span:first-child{font-size:11px}.register-field>input,.password-control input{min-width:0;font-size:14px}.register-field input::placeholder{font-size:12px}.field-hint{font-size:10px}.input-error{font-size:11px;overflow-wrap:anywhere}.register-submit{font-size:11px}.login-prompt{font-size:11px}.return-link{font-size:11px}.register-footer{font-size:10px}@media(max-width:800px){.register-layout{grid-template-columns:minmax(0,1fr);gap:26px}.register-intro h1{font-size:clamp(43px,8vw,56px)}.register-panel{width:100%}}@media(max-width:520px){.register-fields{grid-template-columns:minmax(0,1fr)}.register-field:nth-child(n){grid-column:auto}.register-panel{padding:25px 18px}.register-field>input,.password-control input{font-size:14px}}
</style>
