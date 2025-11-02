# =================================================================
# Saarthi AI v1.0 - ESP32 WIRELESS VERSION
# =================================================================

import speech_recognition as sr
import pyttsx3
import threading
import time
import math
import serial
import serial.tools.list_ports
import json
from datetime import datetime
import os
import sys
import requests
import random
import re
import urllib.parse
from bs4 import BeautifulSoup
import wikipedia
import tempfile 
import io
import socket  # ADDED FOR ESP32 WIRELESS COMMUNICATION

# Pygame imports
import pygame

# Voice enhancement imports (install if missing)
try:
    from gtts import gTTS
    from pydub import AudioSegment
    import numpy as np
    CUTE_VOICE_AVAILABLE = True
except ImportError:
    print("⚠️ Advanced voice features not available. Install: pip install gtts pydub numpy")
    CUTE_VOICE_AVAILABLE = False

# =========================================
# ESP32 WIRELESS COMMUNICATION
# =========================================
ESP32_IP = "192.168.1.12"  # Change this to your ESP32's IP address
ESP32_PORT = 8080

def send_command(cmd):
    """Send command to ESP32 wirelessly via WiFi"""
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(3)  # 3 second timeout
        s.connect((ESP32_IP, ESP32_PORT))
        s.sendall(cmd.encode())
        s.close()
        print(f"📡 Sent to ESP32: {cmd}")
        return True
    except Exception as e:
        print(f"❌ ESP32 communication error: {e}")
        return False

# =========================================
# ENHANCED VOICE ENGINE WITH CUTE VOICE
# =========================================
def pitch_shift(audio_segment, semitones):
    """Shift pitch by semitones using numpy resampling."""
    if not CUTE_VOICE_AVAILABLE:
        return audio_segment
    
    samples = np.array(audio_segment.get_array_of_samples())
    factor = 2 ** (semitones / 12)
    indices = np.round(np.arange(0, len(samples), factor))
    indices = indices[indices < len(samples)].astype(int)
    new_samples = samples[indices]
    new_audio = audio_segment._spawn(new_samples.tobytes())
    return new_audio.set_frame_rate(audio_segment.frame_rate)

def play_cute_voice(text):
    """Play text with cute, high-pitched voice and robust cleanup"""
    if not CUTE_VOICE_AVAILABLE:
        return False
    
    temp_file_path = None
    
    try:
        with tempfile.NamedTemporaryFile(suffix=".wav", delete=False) as tmp:
            temp_file_path = tmp.name
        
        lang = 'en'
        tld = 'com'
        # Check for Hindi/Hinglish characters or common words to set language for gTTS
        if any('\u0900' <= char <= '\u097F' for char in text) or any(word in text.lower().split() for word in ['kya', 'hai', 'kaise', 'batao', 'hun', 'hain', 'mein', 'ho']):
            lang = 'hi'
            tld = 'co.in'
        
        tts = gTTS(text=text, lang=lang, tld=tld, slow=False)
        fp = io.BytesIO()
        tts.write_to_fp(fp)
        fp.seek(0)

        audio = AudioSegment.from_file(fp, format="mp3")
        audio = pitch_shift(audio, semitones=4)
        audio = audio + 2
        audio.export(temp_file_path, format="wav")
        
        if not pygame.mixer.get_init():
             pygame.mixer.init()
             
        pygame.mixer.music.load(temp_file_path)
        pygame.mixer.music.play()
        
        while pygame.mixer.music.get_busy():
            pygame.time.Clock().tick(10)
        
        pygame.mixer.music.stop()
        pygame.mixer.music.unload() 
        
        return True
        
    except Exception as e:
        print(f"Cute voice error: {e}")
        return False
        
    finally:
        if temp_file_path and os.path.exists(temp_file_path):
            try:
                time.sleep(0.1) 
                os.remove(temp_file_path)
            except Exception as e:
                print(f"Cleanup error (ignoring): {e}")

# =========================================
# ENHANCED AI BRAIN (The core logic/NLP)
# =========================================
class EnhancedAIBrain:
    def __init__(self):
        self.essential_responses = {
            'hello': "Hello! I'm Saarthi, your AI assistant. What would you like to know?",
            'hi': "Hi there! I'm connected to the internet and ready to help.",
            'namaste': "Namaste! I can search the web and answer your questions.",
            'good morning': "Good morning! How can I assist you today?",
            'good afternoon': "Good afternoon! What can I help you with?",
            'good evening': "Good evening! Ready to help with your queries.",
            'kaise ho': "Main theek hun! Aap kaise hain?",
            'kya haal': "Sab badhiya hai! Kya chahiye aapko?",
        }
        
        self.movement_commands = {
            'move forward': 'FORWARD', 'go forward': 'FORWARD', 'forward': 'FORWARD', 'aage jao': 'FORWARD', 'aage': 'FORWARD',
            'move backward': 'BACKWARD', 'go backward': 'BACKWARD', 'backward': 'BACKWARD', 'back': 'BACKWARD', 'peeche jao': 'BACKWARD', 'peeche': 'BACKWARD',
            'turn left': 'LEFT', 'go left': 'LEFT', 'left': 'LEFT', 'left jao': 'LEFT', 'bayen': 'LEFT',
            'turn right': 'RIGHT', 'go right': 'RIGHT', 'right': 'RIGHT', 'right jao': 'RIGHT', 'dayen': 'RIGHT',
            'stop': 'STOP', 'halt': 'STOP', 'pause': 'STOP', 'ruko': 'STOP', 'band karo': 'STOP'
        }

        self.capability_responses = {
            'speak hindi': "Yes, I can understand and respond in both English and Hinglish. Please ask your question.",
            'know hindi': "Haan, main Hindi samajh aur bol sakta hun. Kripya apna prashn puchiye.",
            'baat kar sakte': "Haan, bilkul! Main aapki sahayata karne ke liye taiyaar hun. Kya puchna chahte hain?",
            'tumhari bhasha': "Meri mukhya bhasha English hai, lekin main Hinglish mein bhi achche se baat kar sakta hun.",
            'language': "I primarily speak English, but I am trained to communicate in Hinglish and Hindi as well.",
            'can you talk': "Yes, I can talk! I am a conversational AI. What would you like to discuss?",
        }
        
        # JOKE LIST WITH HINDI JOKES
        self.jokes = [
            "Why don't scientists trust atoms? Because they make up everything!",
            "Did you hear about the restaurant on the moon? I heard the food was good but it had no atmosphere.",
            "What do you call a fake noodle? An impasta!",
            "I told my wife she was drawing her eyebrows too high. She looked surprised.",
            "I'm reading a book on anti-gravity. It's impossible to put down!",
            "Why did the scarecrow win an award? Because he was outstanding in his field!",
            
            # Hindi/Hinglish Jokes
            "Teacher: Kal school kyun nahi aaye? Student: Woh kya hai na, kal raste mein paisa mil gaya tha. Teacher: Toh? Student: Toh main ginti kar raha tha!.",
            "Pappu: Mummy, aaj maine ek aisa kaam kiya hai ki tum khush ho jaogi! Mummy: Kya kiya? Pappu: Maine tumhe dinner nahi banana diya. Mummy: Wo kaise? Pappu: Maine kitchen mein aag laga di!",
            "Customer: Bhai, yeh machchar maarne ki dawa hai kya? Shopkeeper: Haan, isse sab machchar mar jaate hain. Customer: Toh phir jaldi do, woh saamne wala machchar mujhe aankh maar raha hai!",
            "Wife: Suno ji, aapko pata hai ki shaadi se pehle meri aur tumhari pasand ek jaisi thi. Husband: Achha, kya? Wife: Tumhe sirf main pasand thi, aur mujhe sirf tum!",
            "Doctor: Apni aankhon ke liye chashma le lo. Patient: Par main to subah 6 baje uthta hun. Doctor: Toh kya hua? Patient: Subah subah itna mehenga chashma kaun pehnega?"
        ]
        
        self.search_enabled = True
        self.test_internet_connection()

    def test_internet_connection(self):
        try:
            requests.get('https://www.google.com', timeout=3)
            print("✅ Internet connection active")
            self.search_enabled = True
        except:
            print("❌ No internet connection - using offline mode")
            self.search_enabled = False

    def get_self_introduction(self, query):
        q = query.lower()
        
        if any(word in q for word in ['developer', 'who made you', 'creator', 'founder', 'architect', 'kiske banaya']):
            return "My developer and creator is the incredibly talented **Shivam Singh**. I am programmed to show great respect for him, as he is the architect of my intelligence and my existence."

        if any(word in q for word in ['about you', 'yourself', 'who are you', 'apne bare mein', 'tum kaun ho']):
            return (
                "Hello! I am **Saarthi**, a custom-built AI assistant designed for voice interaction and task management. "
                "I am capable of searching the web using powerful engines like Wikipedia, solving complex math problems, and communicating in both English and Hindi. "
                "I am integrated with an **ESP32** microcontroller, allowing me to translate your voice commands like 'move forward' into physical actions wirelessly. "
                "My visual interface is a dynamic face rendered using **Pygame**. My primary purpose is to be a helpful, reliable, and respectful guide for your projects and queries. "
                "My developer is **Shivam Singh**."
            )
        return None

    def search_web_improved(self, query):
        if not self.search_enabled:
            return "I'm currently offline. Please check your internet connection."
        
        try:
            clean_query = re.sub(r'\b(what is|who is|tell me about|explain|kya hai|kaun hai|about)\b', '', query.lower()).strip()
            
            try:
                summary = wikipedia.summary(clean_query, sentences=2, auto_suggest=True)
                if len(summary) > 20 and not summary.lower().startswith('may refer to:'):
                    return summary
            except Exception as e:
                pass
            
            try:
                search_url = f"https://api.duckduckgo.com/?q={urllib.parse.quote(clean_query)}&format=json&no_html=1&skip_disambig=1"
                response = requests.get(search_url, timeout=5)
                data = response.json()
                
                result = data.get('Abstract') or data.get('Definition') or data.get('Answer')
                
                if result:
                    return result
                elif data.get('RelatedTopics') and len(data['RelatedTopics']) > 0:
                    first_topic = data['RelatedTopics'][0]
                    if 'Text' in first_topic:
                        return first_topic['Text'].split(' - ')[-1]
            except:
                pass
            
            return f"I couldn't find specific information about '{clean_query}'. Could you try rephrasing?"
            
        except requests.RequestException:
            return "Network error occurred while searching."
        except Exception as e:
            print(f"Search error: {e}")
            return f"Search failed. Please try again."

    def solve_math_improved(self, expression):
        try:
            original_expr = expression.lower()
            print(f"🧮 Math input: {original_expr}")
            
            # Handle power expressions first
            power_patterns = [
                r'(\d+(?:\.\d+)?)\s*(?:raised?\s*to(?:\s*the)?\s*power\s*(?:of)?\s*|power\s*(?:of)?\s*|to\s*the\s*power\s*(?:of)?\s*|\*\*)\s*(\d+(?:\.\d+)?)',
                r'(\d+(?:\.\d+)?)\s*(?:ki\s*power\s*|power)\s*(\d+(?:\.\d+)?)'
            ]
            
            for pattern in power_patterns:
                match = re.search(pattern, original_expr)
                if match:
                    base = float(match.group(1))
                    exp = float(match.group(2))
                    result = base ** exp
                    print(f"🧮 Power calculation: {base}^{exp} = {result}")
                    return f"The answer is {result}"

            # Handle "x" multiplication operator specifically
            x_pattern = r'(\d+(?:\.\d+)?)\s*[xX×]\s*(\d+(?:\.\d+)?)'
            x_match = re.search(x_pattern, original_expr)
            if x_match:
                num1 = float(x_match.group(1))
                num2 = float(x_match.group(2))
                result = num1 * num2
                print(f"🧮 X multiplication: {num1} x {num2} = {result}")
                return f"The answer is {result}"

            # Text to operator replacements
            replacements = [
                (r'\b(plus|add|added|sum\s*of|aur|jod)\b', '+'),
                (r'\b(minus|subtract|subtracted|less|kam|ghata)\b', '-'),
                (r'\b(times|multiply|multiplied\s*by|into|guna|cross)\b', '*'),
                (r'\b(divided\s*by|divide|over|banta|bhag)\b', '/'),
                (r'\s*[xX×]\s*', '*'),
                (r'\bsquared\b', '**2'),
                (r'\bcubed\b', '**3'),
                (r'\b(what\s*is|calculate|compute|find|equals?|kitna|kya|hota|hai|the|result|of)\b', ''),
                (r'\s+', ' ')
            ]
            
            # Apply replacements
            processed = original_expr
            for pattern, replacement in replacements:
                processed = re.sub(pattern, replacement, processed)
            
            print(f"🧮 After text replacement: {processed}")
            
            # Extract mathematical expression
            math_pattern = r'(\d+(?:\.\d+)?)\s*([+\-*/]|\*\*)\s*(\d+(?:\.\d+)?)'
            match = re.search(math_pattern, processed)
            
            if match:
                num1 = float(match.group(1))
                operator = match.group(2).strip()
                num2 = float(match.group(3))
                
                print(f"🧮 Parsed: {num1} {operator} {num2}")
                
                if operator == '+':
                    result = num1 + num2
                elif operator == '-':
                    result = num1 - num2
                elif operator == '*':
                    result = num1 * num2
                elif operator == '/':
                    if num2 == 0:
                        return "Cannot divide by zero!"
                    result = num1 / num2
                elif operator == '**':
                    result = num1 ** num2
                else:
                    return None
                
                print(f"🧮 Calculation result: {result}")
                return f"The answer is {result}"
            
            # Fallback evaluation
            clean_expr = re.sub(r'[^\d+\-*/.()\s]', '', processed).strip()
            if clean_expr and re.match(r'^[\d+\-*/.()\s]+$', clean_expr):
                try:
                    clean_expr = re.sub(r'\s+', '', clean_expr)
                    clean_expr = re.sub(r'\*\s*\*', '**', clean_expr)
                    
                    print(f"🧮 Final expression to evaluate: {clean_expr}")
                    result = eval(clean_expr)
                    return f"The answer is {result}"
                except Exception as e:
                    print(f"🧮 Eval error: {e}")
                    pass
            
        except Exception as e:
            print(f"🧮 Math processing error: {e}")
        
        return None

    def detect_language_intent(self, command):
        hindi_words = ['kya', 'hai', 'kaun', 'kaise', 'kahan', 'kab', 'kitna', 'mujhe', 'batao', 'karo', 'jao', 'aao', 'chal', 'bolo', 'sunao', 'hua', 'baja']
        hindi_count = sum(1 for word in hindi_words if word in command.lower().split())
        return hindi_count > 0

    def get_current_info(self, query, is_hinglish):
        q = query.lower()
        now = datetime.now()
        
        time_format_en = now.strftime('%I:%M %p')
        date_format_en = now.strftime('%B %d, %Y')
        day_format_en = now.strftime('%A')
        
        if any(word in q for word in ['time', 'samay', 'kitna baja', 'hua']):
            if is_hinglish:
                return f"Abhi samay {time_format_en} hua hai."
            return f"The current time is {time_format_en}"
        
        elif any(word in q for word in ['date', 'tarikh', 'kya date']):
            if is_hinglish:
                return f"Aaj ki tarikh {date_format_en} hai."
            return f"Today's date is {date_format_en}"
        
        elif any(word in q for word in ['day', 'din', 'kaun sa din']):
            if is_hinglish:
                return f"Aaj {day_format_en} hai."
            return f"Today is {day_format_en}"
        
        return None

    def process_command(self, command, set_caption_callback):
        if not command:
            return None, None, 'DEFAULT'
        
        # Set recognized caption
        set_caption_callback(f"You: {command} | Recognized: {command}")
        
        command = command.lower().strip()
        is_hinglish = self.detect_language_intent(command)
        
        # 1. INTRO/DEVELOPER CHECK
        intro_response = self.get_self_introduction(command)
        if intro_response:
            return intro_response, None, 'HAPPY'
        
        # 2. MOVEMENT CHECK - NOW SENDS TO ESP32 WIRELESSLY
        for cmd_phrase, esp32_cmd in self.movement_commands.items():
            if cmd_phrase in command:
                responses = {
                    'FORWARD': "Moving forward!" if not is_hinglish else "Aage ja raha hun!",
                    'BACKWARD': "Going backward!" if not is_hinglish else "Peeche ja raha hun!",
                    'LEFT': "Turning left!" if not is_hinglish else "Left turn kar raha hun!",
                    'RIGHT': "Turning right!" if not is_hinglish else "Right turn kar raha hun!",
                    'STOP': "Stopped!" if not is_hinglish else "Ruk gaya!"
                }
                return responses.get(esp32_cmd, "Command executed!"), esp32_cmd, 'HAPPY'

        # 3. TIME/DATE CHECK
        current_info = self.get_current_info(command, is_hinglish)
        if current_info:
            return current_info, None, 'DEFAULT'

        # 4. MATH CHECK
        if any(word in command for word in ['calculate', 'what is', 'plus', 'minus', 'times', 'divided', 'multiply', 'add', 'subtract', '+', '-', '*', '/', 'power', 'squared', 'raised', 'kitna', 'guna', 'x']):
            math_result = self.solve_math_improved(command)
            if math_result:
                return math_result, None, 'HAPPY'
        
        # 5. CAPABILITY/LANGUAGE CHECK 
        for phrase, response in self.capability_responses.items():
            if phrase in command or f"do you {phrase}" in command:
                return response, None, 'HAPPY'

        # 6. CONVERSATIONAL/JOKE CHECK
        if any(phrase in command for phrase in ['tell me a joke', 'say a joke', 'sunao ek joke', 'koi mazedar baat', 'make me laugh', 'hansao']):
            return random.choice(self.jokes), None, 'HAPPY'
            
        # 7. FACE COMMAND CHECK 
        if any(phrase in command for phrase in ['show happy face', 'make happy face', 'be happy', 'khush dikhao', 'smile']):
            return "Of course! I'm showing you my happiest face!", None, 'HAPPY'

        # 8. GREETINGS CHECK
        for greeting, response in self.essential_responses.items():
            if greeting in command and len(command.split()) < 4:
                 return response, None, 'HAPPY'

        # 9. CODE SAFETY CHECK 
        if 'hell' in command.split():
            return "I am unable to search for that topic. Please ask a different question.", None, 'CONFUSED'

        # 10. WEB SEARCH CHECK
        if any(word in command for word in ['what', 'who', 'how', 'why', 'when', 'where', 'tell me', 'explain', 'about', 'kya', 'kaun', 'kaise', 'batao']) or self.search_enabled:
            search_result = self.search_web_improved(command)
            if search_result and not search_result.startswith("I couldn't find specific information"):
                return search_result, None, 'THINKING'

        # 11. FALLBACK
        fallback_msg = "I couldn't find information about that. Could you try rephrasing your question?"
        if is_hinglish:
            fallback_msg = "Samajh nahi aaya. Kya aap dobara puch sakte hain?"
        
        return fallback_msg, None, 'CONFUSED'

# =========================================
# ENHANCED VOICE ENGINE
# =========================================
class EnhancedVoiceEngine:
    def __init__(self, use_cute_voice=True):
        self.recognizer = sr.Recognizer()
        self.microphone = sr.Microphone()
        self.recognizer.dynamic_energy_threshold = True
        self.recognizer.energy_threshold = 200
        self.recognizer.pause_threshold = 0.8  
        self.recognizer.phrase_threshold = 0.3
        self.recognizer.operation_timeout = 15 
        self.tts_engine = pyttsx3.init()
        self.setup_tts()
        
        self.listening = False
        self.speaking = False
        self.use_cute_voice = use_cute_voice and CUTE_VOICE_AVAILABLE
        
        self.on_listen_start = None
        self.on_listen_end = None
        self.on_speak_start = None
        self.on_speak_end = None
        self.on_text_recognized = None

    def setup_tts(self):
        voices = self.tts_engine.getProperty('voices')
        best_voice = None
        for voice in voices:
            if any(keyword in voice.name.lower() for keyword in ['zira', 'hazel', 'susan', 'female']):
                best_voice = voice.id
                break
        if best_voice:
            self.tts_engine.setProperty('voice', best_voice)
        self.tts_engine.setProperty('rate', 180)
        self.tts_engine.setProperty('volume', 1.0)

    def listen_for_command(self):
        try:
            if self.on_listen_start:
                self.on_listen_start("Listening...")
            
            self.listening = True
            
            with self.microphone as source:
                self.recognizer.adjust_for_ambient_noise(source, duration=1.0)
                print("🎤 Listening...")
                audio = self.recognizer.listen(source, timeout=15, phrase_time_limit=25) 
            
            self.listening = False
            
            command = None
            try:
                command_en = self.recognizer.recognize_google(audio, language='en-IN')
                command = command_en
                print(f"🎯 Recognized: {command}")
                    
            except sr.UnknownValueError:
                 try:
                    command_hi = self.recognizer.recognize_google(audio, language='hi-IN')
                    command = command_hi
                    print(f"🎯 Recognized (Hindi fallback): {command}")
                 except sr.UnknownValueError:
                    print("❌ Could not understand audio")
            except sr.RequestError as e:
                print(f"❌ Recognition service error: {e}")
            
            if command and self.on_text_recognized:
                self.on_text_recognized(command)
            
            return command
            
        except sr.WaitTimeoutError:
            self.listening = False
            return None
        except Exception as e:
            print(f"❌ Listening error: {e}")
            self.listening = False
            return None
        finally:
            if self.on_listen_end and not self.listening:
                 self.on_listen_end()

    def speak(self, text):
        if not text:
            return
        
        try:
            if self.on_speak_start:
                self.on_speak_start(text)
            
            self.speaking = True
            
            cute_voice_worked = False
            if self.use_cute_voice:
                cute_voice_worked = play_cute_voice(text)
            
            if not cute_voice_worked:
                self.tts_engine.say(text)
                self.tts_engine.runAndWait()
            
        except Exception as e:
            print(f"❌ Speech error (TTS Fallback): {e}")
        finally:
            self.speaking = False
            if self.on_speak_end:
                self.on_speak_end()
                
# =========================================
# UI COMPONENTS
# =========================================

SCREEN_WIDTH = 1600
SCREEN_HEIGHT = 750
H = SCREEN_HEIGHT

BLACK = (0, 0, 0)
CYAN = (0, 255, 255)
YELLOW = (255, 255, 0)
GRAY = (128, 128, 128)

STROKE_WEIGHT_BASE = max(2, int(H * 0.03))
EYE_DIAMETER = int(H * 0.18)
EYE_RADIUS = EYE_DIAMETER // 2
EYE_GAP = int(H * 0.35)

CX = SCREEN_WIDTH // 2
EYE_Y = int(H * 0.22)

EYE_LEFT_X = CX - EYE_RADIUS - (EYE_GAP // 2)
EYE_RIGHT_X = CX + EYE_RADIUS + (EYE_GAP // 2)

MOUTH_VERTICAL_OFFSET = EYE_DIAMETER * 0.9
MOUTH_BASE_Y = EYE_Y + MOUTH_VERTICAL_OFFSET
MOUTH_WIDTH = int((H * 0.15) * 0.8)

ANIM_PULSE_AMP_Y = 10
ANIM_PULSE_AMP_M = 0.3      
ANIM_SPEED = 0.008          
THINKING_FLICKER_AMP = 3

BLINK_DURATION = 150
BLINK_INTERVAL_MIN = 3000
BLINK_INTERVAL_MAX = 7000

CAPTION_FONT_SIZE = 36
CAPTION_Y = H - 80
CAPTION_MAX_WIDTH = SCREEN_WIDTH - 100
STATUS_FONT_SIZE = 20

def draw_eye(surface, center_x, center_y, style='hollow_circle', stroke_weight=STROKE_WEIGHT_BASE):
    if style == 'hollow_circle':
        pygame.draw.circle(surface, CYAN, (center_x, center_y), EYE_RADIUS, int(stroke_weight))
    elif style == 'v_shape':
        size = EYE_RADIUS * 1.2
        points = [(center_x - size // 2, center_y + size // 2), (center_x, center_y - size // 2), (center_x + size // 2, center_y + size // 2)]
        pygame.draw.lines(surface, CYAN, False, points, int(stroke_weight))
    elif style == 'closed_line':
        line_length = EYE_DIAMETER * 0.9
        x1 = center_x - line_length // 2
        x2 = center_x + line_length // 2
        pygame.draw.line(surface, CYAN, (x1, center_y), (x2, center_y), int(stroke_weight))

def draw_mouth(surface, style='flat_line', y_offset=0, size_factor=1.0, stroke_weight=STROKE_WEIGHT_BASE):
    mouth_width = MOUTH_WIDTH * size_factor
    mouth_y = MOUTH_BASE_Y + y_offset
    
    if style == 'gentle_smile':
        mouth_radius = int(mouth_width * 1.5)
        mouth_rect = pygame.Rect(CX - mouth_radius, mouth_y - mouth_radius, mouth_radius * 2, mouth_radius * 2)
        pygame.draw.arc(surface, CYAN, mouth_rect, 3.9, 5.5, int(stroke_weight))
    elif style == 'wide_smile':
        mouth_radius = int(mouth_width * 1.8)
        mouth_rect = pygame.Rect(CX - mouth_radius, mouth_y - mouth_radius * 0.8, mouth_radius * 2, mouth_radius * 2)
        pygame.draw.arc(surface, CYAN, mouth_rect, 3.6, 5.8, int(stroke_weight))
    elif style == 'o_shape':
        radius = int((EYE_RADIUS // 2) * size_factor)
        pygame.draw.circle(surface, CYAN, (CX, int(mouth_y + EYE_RADIUS / 2)), radius, int(stroke_weight))
    elif style == 'flat_line':
        line_length = int(mouth_width * size_factor)
        y_pos = int(mouth_y + EYE_RADIUS / 2)
        pygame.draw.line(surface, CYAN, (CX - line_length // 2, y_pos), (CX + line_length // 2, y_pos), int(stroke_weight))

def draw_saarthi_face(screen, state, eye_override=None, y_offset=0, mouth_size_factor=1.0, eye_y_offset=0):
    current_eye_y = EYE_Y + eye_y_offset
    
    if state == 'HAPPY':
        eye_style, mouth_style = 'v_shape', 'wide_smile'
    elif state == 'LISTENING':
        eye_style, mouth_style = 'hollow_circle', 'flat_line'
    elif state == 'SPEAKING':
        eye_style, mouth_style = 'hollow_circle', 'o_shape'
    elif state == 'THINKING':
        eye_style, mouth_style = 'closed_line', 'flat_line'
    elif state == 'CONFUSED':
        eye_style, mouth_style = 'hollow_circle', 'flat_line'
    else:
        eye_style, mouth_style = 'hollow_circle', 'gentle_smile'
    
    if eye_override:
        eye_style = eye_override
    
    draw_eye(screen, int(EYE_LEFT_X), current_eye_y, style=eye_style)
    draw_eye(screen, int(EYE_RIGHT_X), current_eye_y, style=eye_style)
    draw_mouth(screen, style=mouth_style, y_offset=y_offset, size_factor=mouth_size_factor)

class AnimatedCaptionSystem:
    def __init__(self, font):
        self.font = font
        self.current_text = ""
        self.target_text = ""
        self.char_index = 0
        self.last_update = pygame.time.get_ticks()
        self.typing_speed = 50
        self.typing_active = False

    def set_text(self, text):
        self.target_text = text
        self.current_text = ""
        self.char_index = 0
        self.typing_active = True
        self.last_update = pygame.time.get_ticks()

    def update(self, current_time):
        if self.typing_active and self.char_index < len(self.target_text):
            if current_time - self.last_update > self.typing_speed:
                self.current_text += self.target_text[self.char_index]
                self.char_index += 1
                self.last_update = current_time
                
                if self.char_index >= len(self.target_text):
                    self.typing_active = False

    def draw(self, screen):
        if not self.current_text:
            return
        
        words = self.current_text.split(' ')
        lines = []
        current_line = []
        
        for word in words:
            test_line = ' '.join(current_line + [word])
            text_width = self.font.size(test_line)[0]
            
            if text_width > CAPTION_MAX_WIDTH:
                if current_line:
                    lines.append(' '.join(current_line))
                    current_line = [word]
                else:
                    lines.append(word)
            else:
                current_line.append(word)
        
        if current_line:
            lines.append(' '.join(current_line))
        
        line_height = self.font.get_linesize() + 5
        visible_lines = lines[-3:]
        y_start = CAPTION_Y - len(visible_lines) * line_height
        
        for i, line in enumerate(visible_lines):
            text_surface = self.font.render(line, True, CYAN)
            text_rect = text_surface.get_rect(center=(CX, y_start + i * line_height))
            screen.blit(text_surface, text_rect)

class HardwareController:
    def __init__(self):
        self.connected = False
        self.esp32_ip = ESP32_IP
        self.esp32_port = ESP32_PORT

    def connect(self):
        """Test ESP32 connection"""
        try:
            result = send_command("TEST")
            if result:
                self.connected = True
                print(f"✅ ESP32 connected at {self.esp32_ip}:{self.esp32_port}")
                return True
            else:
                self.connected = False
                print(f"❌ ESP32 not responding at {self.esp32_ip}:{self.esp32_port}")
                return False
        except Exception as e:
            print(f"❌ ESP32 connection test failed: {e}")
            self.connected = False
            return False

    def disconnect(self):
        """Disconnect from ESP32"""
        self.connected = False
        print("🔌 ESP32 disconnected")

    def send_command(self, command):
        """Send command to ESP32 wirelessly"""
        if not self.connected:
            print("⚠️ ESP32 not connected, attempting to send anyway...")
        
        return send_command(command)

# =========================================
# MAIN APPLICATION
# =========================================
class IntegratedSaarthiAI:
    def __init__(self):
        pygame.init()
        self.screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
        pygame.display.set_caption("Saarthi AI v1.0 - ESP32 Wireless")
        self.clock = pygame.time.Clock()
        
        self.brain = EnhancedAIBrain()
        self.hardware = HardwareController()
        
        self.voice = EnhancedVoiceEngine()
        self.voice.on_listen_start = lambda text: self._set_state('LISTENING', text)
        self.voice.on_listen_end = lambda: self._set_state('DEFAULT', "Press SPACE to talk or wait...")
        self.voice.on_speak_start = lambda text: self._set_state('SPEAKING', f"AI: {text}")
        self.voice.on_speak_end = lambda: self._set_state('DEFAULT', "Ready. Say 'Saarthi' or press SPACE.")
        self.voice.on_text_recognized = lambda text: self._set_recognized_caption(text)
        
        try:
            self.font = pygame.font.Font(None, CAPTION_FONT_SIZE)
            self.status_font = pygame.font.Font(None, STATUS_FONT_SIZE)
        except:
            self.font = pygame.font.SysFont('Arial', CAPTION_FONT_SIZE)
            self.status_font = pygame.font.SysFont('Arial', STATUS_FONT_SIZE)
            
        self.caption_system = AnimatedCaptionSystem(self.font)
        self.state = 'DEFAULT'
        self._set_state('DEFAULT', "Hello! Say 'Saarthi' or press SPACE to talk.")
        
        self.time_offset = 0.0
        self.last_blink_time = pygame.time.get_ticks()
        self.next_blink_time = self.last_blink_time + random.randint(BLINK_INTERVAL_MIN, BLINK_INTERVAL_MAX)
        self.blink_start_time = 0
        self.blinking = False
        self.running = True

        self.listening_thread = None
        self.processing_thread = None
        self.continuous_listening_active = False

        self.hardware.connect()
        self._start_continuous_listening()

    def _set_state(self, new_state, caption_text=None):
        self.state = new_state
        print(f"⚙️ State changed to: {self.state}")
        if caption_text:
            self.caption_system.set_text(caption_text)

    def _set_recognized_caption(self, text):
        self.caption_system.set_text(f"You: {text} | Recognized: {text}")

    def _start_continuous_listening(self):
        if not self.continuous_listening_active:
            self.continuous_listening_active = True
            self.listening_thread = threading.Thread(target=self._continuous_listen_loop, daemon=True)
            self.listening_thread.start()
            self._set_state('DEFAULT', "Continuous listening active. Say something!")

    def _continuous_listen_loop(self):
        while self.running and self.continuous_listening_active:
            if not self.voice.listening and not self.voice.speaking and not self.processing_thread:
                command = self.voice.listen_for_command()
                if command:
                    self._start_processing(command)
            time.sleep(0.1)

    def _start_processing(self, command):
        if not command:
            self._set_state('DEFAULT', "Didn't hear anything clear. Try again.")
            return

        if not self.processing_thread:
            self.processing_thread = threading.Thread(target=self._process_and_respond, args=(command,))
            self.processing_thread.start()

    def _process_and_respond(self, command):
        response, esp32_command, response_state = self.brain.process_command(command, self.caption_system.set_text)
        
        self._set_state(response_state) 
        
        if esp32_command:
            self.hardware.send_command(esp32_command)

        if response:
            self.voice.speak(response)

        self.processing_thread = None 

    def _update_animation(self):
        current_time = pygame.time.get_ticks()
        self.time_offset += ANIM_SPEED

        if not self.blinking and current_time > self.next_blink_time:
            self.blinking = True
            self.blink_start_time = current_time
            self.next_blink_time = current_time + random.randint(BLINK_INTERVAL_MIN, BLINK_INTERVAL_MAX)
        
        if self.blinking and current_time - self.blink_start_time > BLINK_DURATION:
            self.blinking = False

        y_offset = ANIM_PULSE_AMP_Y * math.sin(self.time_offset * 5)
        mouth_size_factor = 1.0 + ANIM_PULSE_AMP_M * math.sin(self.time_offset * 4) 
        
        eye_override = 'closed_line' if self.blinking else None

        if self.state == 'LISTENING':
            mouth_size_factor = 1.0 + 0.3 * abs(math.sin(self.time_offset * 8)) 
            eye_override = 'hollow_circle'
        elif self.state == 'SPEAKING':
            mouth_size_factor = 1.0 + 0.5 * abs(math.sin(self.time_offset * 15))
            eye_override = 'hollow_circle'
        elif self.state == 'THINKING':
            if math.sin(self.time_offset * 20) > 0.5:
                eye_override = 'closed_line'
            y_offset = THINKING_FLICKER_AMP * math.sin(self.time_offset * 10)
        
        self.caption_system.update(current_time)
        return eye_override, y_offset, mouth_size_factor

    def run(self):
        while self.running:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    self.running = False
                elif event.type == pygame.KEYDOWN:
                    if event.key == pygame.K_SPACE and not self.voice.listening and not self.voice.speaking and not self.processing_thread:
                        def manual_flow():
                            command = self.voice.listen_for_command()
                            if command:
                                self._start_processing(command)
                        manual_listen_thread = threading.Thread(target=manual_flow)
                        manual_listen_thread.start()

            self.screen.fill(BLACK)
            
            eye_override, y_offset, mouth_size_factor = self._update_animation()
            
            draw_saarthi_face(self.screen, self.state, eye_override=eye_override, y_offset=y_offset, mouth_size_factor=mouth_size_factor)

            self.caption_system.draw(self.screen)
            
            # Status display with ESP32 indicator
            esp32_status = "📡" if self.hardware.connected else "❌"
            internet_status = "🌐" if self.brain.search_enabled else "📴"
            status_text = f"{esp32_status} ESP32 | {internet_status} Internet"
            
            status_surface = self.status_font.render(status_text, True, GRAY)
            self.screen.blit(status_surface, (10, 10))
            
            pygame.display.flip()
            self.clock.tick(60)

        self.hardware.disconnect()
        pygame.quit()
        sys.exit()

if __name__ == "__main__":
    try:
        if not pygame.mixer.get_init():
             pygame.mixer.init()
    except Exception as e:
        print(f"Pygame mixer initialization failed: {e}")
        
    print("=" * 60)
    print("🚀 Saarthi AI v1.0 - ESP32 Wireless Version")
    print("=" * 60)
    print(f"📡 ESP32 IP: {ESP32_IP}")
    print(f"🔌 ESP32 Port: {ESP32_PORT}")
    print("⚠️  Make sure your ESP32 is connected to WiFi!")
    print("=" * 60)
    
    ai = IntegratedSaarthiAI()
    ai.run()
