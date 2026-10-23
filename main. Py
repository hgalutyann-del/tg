import os
import io
import json
import asyncio
import threading
from datetime import datetime

from kivy.app import App
from kivy.lang import Builder
from kivy.clock import Clock
from kivy.core.window import Window
from kivy.utils import platform
from kivy.properties import StringProperty, BooleanProperty
from kivy.uix.screenmanager import ScreenManager, Screen, SlideTransition
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.popup import Popup

from telethon import TelegramClient, events
from telethon.tl.functions.messages import ReadMessageContentsRequest

Window.clearcolor = (0, 0, 0, 1)

KV = """
<IOSButton@Button>:
    background_normal: ''
    background_color: (0, 0.48, 1, 1)
    color: (1, 1, 1, 1)
    font_size: '16sp'
    bold: True
    size_hint_y: None
    height: '46dp'

<AuthInput@TextInput>:
    multiline: False
    background_normal: ''
    background_active: ''
    background_color: (0.11, 0.11, 0.12, 1)
    foreground_color: (1, 1, 1, 1)
    hint_text_color: (0.55, 0.55, 0.58, 1)
    cursor_color: (0, 0.48, 1, 1)
    font_size: '16sp'
    padding: ['14dp', '12dp']
    size_hint_y: None
    height: '46dp'

<LoginScreen>:
    BoxLayout:
        orientation: 'vertical'
        padding: '24dp'
        spacing: '14dp'
        
        Label:
            text: 'Telegram iOS Mod'
            font_size: '24sp'
            bold: True
            size_hint_y: None
            height: '50dp'
            
        Label:
            id: status_lbl
            text: 'Введите api_id, api_hash и телефон'
            color: (0.55, 0.55, 0.58, 1)
            font_size: '14sp'
            size_hint_y: None
            height: '30dp'

        AuthInput:
            id: api_id_in
            hint_text: 'API ID (цифры с my.telegram.org)'
            input_filter: 'int'

        AuthInput:
            id: api_hash_in
            hint_text: 'API Hash (строка)'

        AuthInput:
            id: phone_in
            hint_text: 'Номер телефона (+7... или +374...)'

        AuthInput:
            id: code_in
            hint_text: 'Код из Telegram (после запроса)'
            disabled: True

        AuthInput:
            id: pass_in
            hint_text: 'Облачный пароль 2FA (если есть)'
            password: True
            disabled: True

        IOSButton:
            id: action_btn
            text: 'Получить код'
            on_release: app.handle_auth()
            
        Widget:

<ChatsScreen>:
    BoxLayout:
        orientation: 'vertical'
        
        BoxLayout:
            size_hint_y: None
            height: '54dp'
            padding: ['12dp', '6dp']
            canvas.before:
                Color:
                    rgba: (0.09, 0.09, 0.1, 1)
                Rectangle:
                    pos: self.pos
                    size: self.size
            Label:
                text: 'Чаты (iOS Mod)'
                font_size: '18sp'
                bold: True
                halign: 'left'
            Button:
                text: 'Обновить'
                size_hint_x: None
                width: '90dp'
                background_normal: ''
                background_color: (0.18, 0.18, 0.2, 1)
                on_release: app.load_dialogs()

        ScrollView:
            GridLayout:
                id: dialogs_box
                cols: 1
                size_hint_y: None
                height: self.minimum_height
                spacing: '1dp'

        BoxLayout:
            size_hint_y: None
            height: '54dp'
            canvas.before:
                Color:
                    rgba: (0.09, 0.09, 0.1, 1)
                Rectangle:
                    pos: self.pos
                    size: self.size
            Button:
                text: 'Чаты'
                background_normal: ''
                background_color: (0, 0, 0, 0)
                color: (0, 0.48, 1, 1)
            Button:
                text: 'Настройки мода'
                background_normal: ''
                background_color: (0, 0, 0, 0)
                color: (0.55, 0.55, 0.58, 1)
                on_release: app.root.current = 'settings'

<ChatViewScreen>:
    BoxLayout:
        orientation: 'vertical'
        
        BoxLayout:
            size_hint_y: None
            height: '54dp'
            padding: ['8dp', '6dp']
            spacing: '8dp'
            canvas.before:
                Color:
                    rgba: (0.09, 0.09, 0.1, 1)
                Rectangle:
                    pos: self.pos
                    size: self.size
            Button:
                text: '< Чаты'
                size_hint_x: None
                width: '75dp'
                background_normal: ''
                background_color: (0, 0, 0, 0)
                color: (0, 0.48, 1, 1)
                font_size: '16sp'
                on_release: app.back_to_chats()
            Label:
                id: chat_title
                text: 'Чат'
                font_size: '17sp'
                bold: True
                shorten: True
            Button:
                text: 'Обн.'
                size_hint_x: None
                width: '55dp'
                background_normal: ''
                background_color: (0.18, 0.18, 0.2, 1)
                on_release: app.load_messages(app.current_chat_id, app.current_chat_name)

        ScrollView:
            id: msg_scroll
            GridLayout:
                id: messages_box
                cols: 1
                size_hint_y: None
                height: self.minimum_height
                padding: '10dp'
                spacing: '8dp'

        BoxLayout:
            size_hint_y: None
            height: '54dp'
            padding: ['8dp', '6dp']
            spacing: '8dp'
            canvas.before:
                Color:
                    rgba: (0.09, 0.09, 0.1, 1)
                Rectangle:
                    pos: self.pos
                    size: self.size
            AuthInput:
                id: msg_input
                hint_text: 'Сообщение...'
                height: '40dp'
            Button:
                text: 'Отпр'
                size_hint_x: None
                width: '65dp'
                background_normal: ''
                background_color: (0, 0.48, 1, 1)
                bold: True
                on_release: app.send_msg()

<SettingsScreen>:
    BoxLayout:
        orientation: 'vertical'
        padding: '16dp'
        spacing: '12dp'
        
        Label:
            text: 'Настройки iOS Мода'
            font_size: '20sp'
            bold: True
            size_hint_y: None
            height: '44dp'

        BoxLayout:
            size_hint_y: None
            height: '50dp'
            Label:
                text: 'Режим невидимки (Не читать)'
                halign: 'left'
            CheckBox:
                active: app.ghost_mode
                on_active: app.ghost_mode = self.active

        BoxLayout:
            size_hint_y: None
            height: '50dp'
            Label:
                text: 'Сохранять удаленные (Анти-удаление)'
                halign: 'left'
            CheckBox:
                active: app.keep_deleted
                on_active: app.keep_deleted = self.active

        BoxLayout:
            size_hint_y: None
            height: '50dp'
            Label:
                text: 'Показывать историю изменений'
                halign: 'left'
            CheckBox:
                active: app.show_edits
                on_active: app.show_edits = self.active

        Widget:

        IOSButton:
            text: 'Вернуться к чатам'
            on_release: app.root.current = 'chats'
"""

class LoginScreen(Screen): pass
class ChatsScreen(Screen): pass
class ChatViewScreen(Screen): pass
class SettingsScreen(Screen): pass


class TelegramModApp(App):
    ghost_mode = BooleanProperty(True)
    keep_deleted = BooleanProperty(True)
    show_edits = BooleanProperty(True)

    def build(self):
        Builder.load_string(KV)
        self.sm = ScreenManager(transition=SlideTransition())
        self.sm.add_widget(LoginScreen(name='login'))
        self.sm.add_widget(ChatsScreen(name='chats'))
        self.sm.add_widget(ChatViewScreen(name='chat_view'))
        self.sm.add_widget(SettingsScreen(name='settings'))

        self.data_dir = self.user_data_dir
        self.config_path = os.path.join(self.data_dir, "config.json")
        self.session_path = os.path.join(self.data_dir, "mod_session")

        self.client = None
        self.phone = ""
        self.current_chat_id = None
        self.current_chat_name = ""
        self.cached_dialogs = []
        self.history_cache = {}
        self.auth_step = "request_code"

        self.loop = asyncio.new_event_loop()
        threading.Thread(target=self._start_async_loop, daemon=True).start()

        Clock.schedule_once(lambda dt: self.check_existing_session(), 0.5)
        return self.sm

    def _start_async_loop(self):
        asyncio.set_event_loop(self.loop)
        self.loop.run_forever()

    def run_coro(self, coro):
        return asyncio.run_coroutine_threadsafe(coro, self.loop)

    def check_existing_session(self):
        if os.path.exists(self.config_path):
            try:
                with open(self.config_path, "r", encoding="utf-8") as f:
                    cfg = json.load(f)
                scr = self.sm.get_screen('login')
                scr.ids.api_id_in.text = str(cfg.get("api_id", ""))
                scr.ids.api_hash_in.text = cfg.get("api_hash", "")
                scr.ids.phone_in.text = cfg.get("phone", "")
                self.run_coro(self._auto_connect(cfg["api_id"], cfg["api_hash"]))
            except Exception:
                pass

    async def _auto_connect(self, api_id, api_hash):
        self.client = TelegramClient(self.session_path, int(api_id), api_hash, loop=self.loop)
        await self.client.connect()
        if await self.client.is_user_authorized():
            self._register_handlers()
            Clock.schedule_once(lambda dt: self._go_to_chats(), 0)

    def handle_auth(self):
        scr = self.sm.get_screen('login')
        api_id = scr.ids.api_id_in.text.strip()
        api_hash = scr.ids.api_hash_in.text.strip()
        phone = scr.ids.phone_in.text.strip()
        code = scr.ids.code_in.text.strip()
        password = scr.ids.pass_in.text.strip()

        if not api_id or not api_hash or not phone:
            scr.ids.status_lbl.text = "Заполните API ID, Hash и телефон!"
            return

        if self.auth_step == "request_code":
            scr.ids.status_lbl.text = "Отправка кода в Telegram..."
            self.run_coro(self._send_code(int(api_id), api_hash, phone))
        else:
            scr.ids.status_lbl.text = "Проверка кода..."
            self.run_coro(self._sign_in(int(api_id), api_hash, phone, code, password))

    async def _send_code(self, api_id, api_hash, phone):
        try:
            if not self.client:
                self.client = TelegramClient(self.session_path, api_id, api_hash, loop=self.loop)
            await self.client.connect()
            await self.client.send_code_request(phone)
            self.auth_step = "verify_code"
            def _ui():
                scr = self.sm.get_screen('login')
                scr.ids.code_in.disabled = False
                scr.ids.pass_in.disabled = False
                scr.ids.action_btn.text = "Войти в аккаунт"
                scr.ids.status_lbl.text = "Код отправлен! Введите его ниже:"
            Clock.schedule_once(lambda dt: _ui(), 0)
        except Exception as e:
            Clock.schedule_once(lambda dt: setattr(self.sm.get_screen('login').ids.status_lbl, 'text', f"Ошибка: {e}"), 0)

    async def _sign_in(self, api_id, api_hash, phone, code, password):
        try:
            try:
                await self.client.sign_in(phone=phone, code=code)
            except Exception:
                if password:
                    await self.client.sign_in(password=password)
                else:
                    raise
            with open(self.config_path, "w", encoding="utf-8") as f:
                json.dump({"api_id": api_id, "api_hash": api_hash, "phone": phone}, f)
            self._register_handlers()
            Clock.schedule_once(lambda dt: self._go_to_chats(), 0)
        except Exception as e:
            Clock.schedule_once(lambda dt: setattr(self.sm.get_screen('login').ids.status_lbl, 'text', f"Ошибка входа: {e}"), 0)

    def _register_handlers(self):
        @self.client.on(events.NewMessage)
        async def on_new(event):
            m = event.message
            self.history_cache[m.id] = {
                "id": m.id, "chat_id": event.chat_id,
                "text": m.raw_text or "[Медиафайл]", "orig": None,
                "deleted": False, "out": m.out,
                "time": m.date.strftime("%H:%M") if m.date else ""
            }
            if self.current_chat_id == event.chat_id:
                await self._fetch_messages(self.current_chat_id)

        @self.client.on(events.MessageEdited)
        async def on_edit(event):
            if not self.show_edits: return
            if event.id in self.history_cache:
                old = self.history_cache[event.id]["text"]
                if old != event.raw_text:
                    self.history_cache[event.id]["orig"] = old
                    self.history_cache[event.id]["text"] = event.raw_text
            if self.current_chat_id == event.chat_id:
                await self._fetch_messages(self.current_chat_id)

        @self.client.on(events.MessageDeleted)
        async def on_del(event):
            if not self.keep_deleted: return
            for mid in event.deleted_ids:
                if mid in self.history_cache:
                    self.history_cache[mid]["deleted"] = True
            if self.current_chat_id:
                await self._fetch_messages(self.current_chat_id)

    def _go_to_chats(self):
        self.sm.current = 'chats'
        self.load_dialogs()

    def load_dialogs(self):
        self.run_coro(self._fetch_dialogs())

    async def _fetch_dialogs(self):
        dialogs = await self.client.get_dialogs(limit=35)
        res = []
        for d in dialogs:
            res.append({
                "id": d.id,
                "name": d.name or "Без имени",
                "unread": d.unread_count,
                "noforwards": bool(getattr(d.entity, "noforwards", False)),
                "last": (d.message.message if d.message and d.message.message else "[Медиа]")
            })
        self.cached_dialogs = res
        Clock.schedule_once(lambda dt: self._render_dialogs(res), 0)

    def _render_dialogs(self, dialogs):
        box = self.sm.get_screen('chats').ids.dialogs_box
        box.clear_widgets()
        for d in dialogs:
            lock = " [Обход пересылки]" if d["noforwards"] else ""
            unread = f" ({d['unread']})" if d["unread"] > 0 else ""
            btn = Button(
                text=f"{d['name']}{lock}{unread}\n{d['last'][:40]}",
                size_hint_y=None, height='68dp',
                halign='left', valign='middle',
                background_normal='', background_color=(0.11, 0.11, 0.12, 1),
                color=(1, 1, 1, 1), font_size='15sp'
            )
            btn.bind(size=lambda s, w: setattr(s, 'text_size', (w[0] - 24, None)))
            btn.bind(on_release=lambda x, cid=d["id"], cname=d["name"]: self.open_chat(cid, cname))
            box.add_widget(btn)

    def open_chat(self, chat_id, chat_name):
        self.current_chat_id = chat_id
        self.current_chat_name = chat_name
        self.sm.get_screen('chat_view').ids.chat_title.text = chat_name
        self.sm.current = 'chat_view'
        self.load_messages(chat_id, chat_name)

    def back_to_chats(self):
        self.current_chat_id = None
        self.sm.current = 'chats'

    def load_messages(self, chat_id, chat_name):
        self.run_coro(self._fetch_messages(chat_id))

    async def _fetch_messages(self, chat_id):
        if not self.ghost_mode:
            try: await self.client.send_read_acknowledge(chat_id)
            except Exception: pass

        msgs = await self.client.get_messages(chat_id, limit=30)
        items = []
        seen = set()

        for m in msgs:
            seen.add(m.id)
            cached = self.history_cache.get(m.id, {})
            is_ttl = bool(m.media and getattr(m.media, "ttl_seconds", None))
            txt = m.raw_text or ("[Одноразовое фото 🔥]" if is_ttl else ("[Медиафайл]" if m.media else ""))
            items.append({
                "id": m.id, "text": txt,
                "orig": cached.get("orig"),
                "deleted": cached.get("deleted", False),
                "noforwards": bool(getattr(m, "noforwards", False)) or is_ttl,
                "out": m.out,
                "time": m.date.strftime("%H:%M") if m.date else ""
            })

        if self.keep_deleted:
            for mid, c in self.history_cache.items():
                if c["chat_id"] == chat_id and c["deleted"] and mid not in seen:
                    items.append({
                        "id": mid, "text": c["text"],
                        "orig": c.get("orig"), "deleted": True,
                        "noforwards": False, "out": c["out"], "time": c["time"]
                    })

        items.sort(key=lambda x: x["id"])
        Clock.schedule_once(lambda dt: self._render_messages(items), 0)

    def _render_messages(self, items):
        box = self.sm.get_screen('chat_view').ids.messages_box
        box.clear_widgets()
        for m in items:
            prefix = ""
            if m["deleted"]: prefix += "[УДАЛЕНО СОБЕСЕДНИКОМ]\n"
            if m["orig"]: prefix += f"[До ред.: {m['orig']}]\n"
            if m["noforwards"]: prefix += "[Защита снята 🔓]\n"

            bg = (0, 0.48, 1, 1) if m["out"] else (0.15, 0.15, 0.17, 1)
            btn = Button(
                text=f"{prefix}{m['text']}\n({m['time']}) [Нажми для пересылки]",
                size_hint_y=None, height='74dp',
                halign='left', valign='middle',
                background_normal='', background_color=bg,
                color=(1, 1, 1, 1), font_size='14sp'
            )
            btn.bind(size=lambda s, w: setattr(s, 'text_size', (w[0] - 24, None)))
            btn.bind(on_release=lambda x, mid=m["id"]: self.show_forward_popup(mid))
            box.add_widget(btn)

    def send_msg(self):
        inp = self.sm.get_screen('chat_view').ids.msg_input
        text = inp.text.strip()
        if not text or not self.current_chat_id: return
        inp.text = ""
        self.run_coro(self._async_send(self.current_chat_id, text))

    async def _async_send(self, chat_id, text):
        await self.client.send_message(chat_id, text)
        await self._fetch_messages(chat_id)

    def show_forward_popup(self, msg_id):
        layout = BoxLayout(orientation='vertical', spacing='6dp', padding='10dp')
        for d in self.cached_dialogs[:10]:
            b = Button(text=d["name"], size_hint_y=None, height='42dp')
            b.bind(on_release=lambda x, tid=d["id"]: self._start_bypass(msg_id, tid, popup))
            layout.add_widget(b)
        popup = Popup(title='Переслать (Обход запрета) в:', content=layout, size_hint=(0.9, 0.75))
        popup.open()

    def _start_bypass(self, msg_id, target_id, popup):
        popup.dismiss()
        self.run_coro(self._bypass_forward(self.current_chat_id, msg_id, target_id))

    async def _bypass_forward(self, from_chat, msg_id, to_chat):
        msg = await self.client.get_messages(from_chat, ids=msg_id)
        if not msg: return
        if msg.media:
            raw = await self.client.download_media(msg, file=bytes)
            if raw:
                stream = io.BytesIO(raw)
                stream.name = "bypass_media.jpg"
                await self.client.send_file(to_chat, file=stream, caption=msg.raw_text or "")
                return
        await self.client.send_message(to_chat, msg.raw_text or "[Пусто]")

if __name__ == '__main__':
    TelegramModApp().run()
