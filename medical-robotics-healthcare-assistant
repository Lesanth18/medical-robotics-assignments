import browserbotics as bb
import time
import math
import random

# ═══════════════════════════════════════════════════════════════
#   BROWSERBOTICS — NURSE ROBOT v6
#   Clinic room: single bed, TV, window with curtains (open/close)
#   Nurse robot checks oxygen level & glucose level
#   Robot looks like a NURSE (white uniform, cap, apron)
# ═══════════════════════════════════════════════════════════════

TICK = 0.022

# ── Room layout ──────────────────────────────────────────────
# Room is 7 x 9 units
# Bed at right-centre
# TV on far wall (back)
# Window on RIGHT wall with curtains
# Door on LEFT wall

ROOM_W  =  7.0   # X axis
ROOM_D  =  9.0   # Y axis
ROOM_H  =  3.2   # Z axis

# Key world positions
START_X,  START_Y  = -3.0,  0.0      # nurse start (near door)
BED_CX,   BED_CY   =  1.8,  0.5      # bed centre
PATIENT_HEAD = (1.8, 1.35, 0.72)     # patient head world pos

# Nurse stands beside bed for checks
CHECK_X,  CHECK_Y  =  0.7,  0.80     # stand here, face patient
# Nurse faces patient head from CHECK pos
def mouth_yaw():
    return math.atan2(PATIENT_HEAD[1]-CHECK_Y, PATIENT_HEAD[0]-CHECK_X)

# Window on RIGHT wall: X = ROOM_W/2 = 3.5
WIN_X  =  3.5 - 0.05   # right wall inner face
WIN_CY =  0.5           # window centre Y
WIN_Z  =  1.65          # window centre Z

# Curtain panels — LEFT panel moves from closed(WIN_CY-0.0) to open(WIN_CY-0.55)
CURTAIN_L_CLOSED_Y =  WIN_CY + 0.00
CURTAIN_L_OPEN_Y   =  WIN_CY - 0.60
CURTAIN_R_CLOSED_Y =  WIN_CY + 0.00
CURTAIN_R_OPEN_Y   =  WIN_CY + 0.60

# TV on back wall
TV_X, TV_Y, TV_Z = 0.0, -ROOM_D/2+0.10, 1.55

# ── helper math ──────────────────────────────────────────────
def ease(t):
    t = max(0.0, min(1.0, t))
    return t*t*(3-2*t)

def lerp(a, b, t):
    return a + (b-a)*t

def norm_angle(a):
    while a >  math.pi: a -= 2*math.pi
    while a < -math.pi: a += 2*math.pi
    return a

def face_yaw(fx, fy, tx, ty):
    return math.atan2(ty-fy, tx-fx)

def place(bid, x, y, z, rx=0, ry=0, rz=0):
    bb.resetBasePose(bid, [x,y,z], bb.getQuaternionFromEuler([rx,ry,rz]))

# ═══════════════════════════════════════════════════════════════
#   CLINIC ROOM
# ═══════════════════════════════════════════════════════════════
def build_clinic():
    bb.addGroundPlane()
    hw = ROOM_W/2
    hd = ROOM_D/2

    # ── Walls ─────────────────────────────────────────────────
    # Left wall (door side)
    bb.createBody('box', halfExtent=[0.06, hd, ROOM_H/2],
                  position=[-hw, 0, ROOM_H/2], color='#f5f0eb', mass=0)
    # Right wall (window side) — split into 3 panels around window
    for py, ext in [
        (-hd/2-0.5, [0.06, hd/2-0.85, ROOM_H/2]),   # below window zone
        ( hd/2+0.5, [0.06, hd/2-0.85, ROOM_H/2]),   # above window zone
    ]:
        bb.createBody('box', halfExtent=ext,
                      position=[hw, py, ROOM_H/2], color='#f5f0eb', mass=0)
    # Right wall above and below window opening
    bb.createBody('box', halfExtent=[0.06, 0.90, (WIN_Z-0.65)/2],
                  position=[hw, WIN_CY, (WIN_Z-0.65)/2], color='#f5f0eb', mass=0)
    bb.createBody('box', halfExtent=[0.06, 0.90, (ROOM_H-(WIN_Z+0.65))/2],
                  position=[hw, WIN_CY, WIN_Z+0.65+(ROOM_H-(WIN_Z+0.65))/2],
                  color='#f5f0eb', mass=0)

    # Back wall (TV wall)
    bb.createBody('box', halfExtent=[hw, 0.06, ROOM_H/2],
                  position=[0, -hd, ROOM_H/2], color='#ede8e2', mass=0)
    # Front wall
    bb.createBody('box', halfExtent=[hw, 0.06, ROOM_H/2],
                  position=[0,  hd, ROOM_H/2], color='#f5f0eb', mass=0)

    # Ceiling
    bb.createBody('box', halfExtent=[hw, hd, 0.05],
                  position=[0, 0, ROOM_H+0.05], color='white', mass=0)

    # ── Floor — warm cream tiles ──────────────────────────────
    for ix in range(-3, 4):
        for iy in range(-4, 5):
            c = '#e8e0d5' if (ix+iy)%2==0 else '#f0ebe3'
            bb.createBody('box', halfExtent=[0.494,0.494,0.014],
                          position=[ix+0.5, iy+0.5, 0.014], color=c, mass=0)

    # ── Skirting ──────────────────────────────────────────────
    for pos, ext in [
        ([-hw,0,0.07],[0.02,hd,0.07]),
        ([ hw,0,0.07],[0.02,hd,0.07]),
        ([0,-hd,0.07],[hw,0.02,0.07]),
        ([0, hd,0.07],[hw,0.02,0.07]),
    ]:
        bb.createBody('box', halfExtent=ext, position=pos,
                      color='#c8bfb0', mass=0)

    # ── Ceiling light ─────────────────────────────────────────
    bb.createBody('box', halfExtent=[0.60, 0.18, 0.04],
                  position=[0, 0, ROOM_H-0.02], color='#fffde7', mass=0)
    # Light housing
    bb.createBody('box', halfExtent=[0.65, 0.23, 0.035],
                  position=[0, 0, ROOM_H], color='#e0ddd8', mass=0)

    # ── Door (left wall) ──────────────────────────────────────
    bb.createBody('box', halfExtent=[0.05, 0.48, 1.08],
                  position=[-hw+0.02, -2.0, 1.08], color='#c8a87a', mass=0)
    # Door frame
    for pos, ext in [
        ([-hw, -2.0, 2.20],  [0.08, 0.52, 0.06]),   # top
        ([-hw, -2.50, 1.08], [0.08, 0.04, 1.10]),   # left post
        ([-hw, -1.50, 1.08], [0.08, 0.04, 1.10]),   # right post
    ]:
        bb.createBody('box', halfExtent=ext, position=pos,
                      color='#a07850', mass=0)
    # Door handle
    bb.createBody('sphere', radius=0.040,
                  position=[-hw+0.08, -1.56, 1.05], color='#d4af37', mass=0)

    print("[CLINIC] Room built ✔")


# ═══════════════════════════════════════════════════════════════
#   WINDOW + CURTAINS (animated)
# ═══════════════════════════════════════════════════════════════
class Window:
    def __init__(self):
        self.curtain_l = None
        self.curtain_r = None
        self._build()

    def _build(self):
        # Window glass
        bb.createBody('box', halfExtent=[0.04, 0.88, 0.64],
                      position=[WIN_X, WIN_CY, WIN_Z],
                      color='#aed6f1', mass=0)
        # Window frame
        for pos, ext in [
            ([WIN_X, WIN_CY, WIN_Z+0.66], [0.06, 0.92, 0.04]),  # top bar
            ([WIN_X, WIN_CY, WIN_Z-0.66], [0.06, 0.92, 0.04]),  # bottom bar
            ([WIN_X, WIN_CY+0.90, WIN_Z], [0.06, 0.04, 0.68]),  # right post
            ([WIN_X, WIN_CY-0.90, WIN_Z], [0.06, 0.04, 0.68]),  # left post
            ([WIN_X, WIN_CY,      WIN_Z], [0.04, 0.02, 0.68]),  # centre vertical
            ([WIN_X, WIN_CY,      WIN_Z], [0.04, 0.90, 0.02]),  # centre horizontal
        ]:
            bb.createBody('box', halfExtent=ext, position=pos,
                          color='white', mass=0)

        # Curtain rod
        bb.createBody('box', halfExtent=[0.015, 1.05, 0.015],
                      position=[WIN_X-0.08, WIN_CY, WIN_Z+0.82],
                      color='#8b7355', mass=0)
        # Rod finials
        for dy in [-1.08, 1.08]:
            bb.createBody('sphere', radius=0.030,
                          position=[WIN_X-0.08, WIN_CY+dy, WIN_Z+0.82],
                          color='#d4af37', mass=0)

        # Curtain panels — start CLOSED (covering window)
        self.curtain_l = bb.createBody('box',
            halfExtent=[0.04, 0.44, 0.65],
            position=[WIN_X-0.06, CURTAIN_L_CLOSED_Y-0.22, WIN_Z],
            color='#e8c4a0', mass=0)
        self.curtain_r = bb.createBody('box',
            halfExtent=[0.04, 0.44, 0.65],
            position=[WIN_X-0.06, CURTAIN_R_CLOSED_Y+0.22, WIN_Z],
            color='#e8c4a0', mass=0)

        # Curtain pattern details
        for dz in [-0.3, 0, 0.3]:
            bb.createBody('box', halfExtent=[0.005, 0.42, 0.015],
                          position=[WIN_X-0.025, WIN_CY-0.22, WIN_Z+dz],
                          color='#c8a070', mass=0)
            bb.createBody('box', halfExtent=[0.005, 0.42, 0.015],
                          position=[WIN_X-0.025, WIN_CY+0.22, WIN_Z+dz],
                          color='#c8a070', mass=0)

        print("[WINDOW] Window + curtains built ✔")

    def animate_open(self, steps=45):
        """Slide curtains open smoothly."""
        cl_start = CURTAIN_L_CLOSED_Y - 0.22
        cl_end   = CURTAIN_L_OPEN_Y   - 0.22
        cr_start = CURTAIN_R_CLOSED_Y + 0.22
        cr_end   = CURTAIN_R_OPEN_Y   + 0.22
        for i in range(steps+1):
            t  = ease(i/steps)
            ly = lerp(cl_start, cl_end, t)
            ry = lerp(cr_start, cr_end, t)
            place(self.curtain_l, WIN_X-0.06, ly, WIN_Z)
            place(self.curtain_r, WIN_X-0.06, ry, WIN_Z)
            time.sleep(TICK*1.5)

    def animate_close(self, steps=45):
        """Slide curtains closed smoothly."""
        cl_start = CURTAIN_L_OPEN_Y   - 0.22
        cl_end   = CURTAIN_L_CLOSED_Y - 0.22
        cr_start = CURTAIN_R_OPEN_Y   + 0.22
        cr_end   = CURTAIN_R_CLOSED_Y + 0.22
        for i in range(steps+1):
            t  = ease(i/steps)
            ly = lerp(cl_start, cl_end, t)
            ry = lerp(cr_start, cr_end, t)
            place(self.curtain_l, WIN_X-0.06, ly, WIN_Z)
            place(self.curtain_r, WIN_X-0.06, ry, WIN_Z)
            time.sleep(TICK*1.5)


# ═══════════════════════════════════════════════════════════════
#   TV ON BACK WALL
# ═══════════════════════════════════════════════════════════════
def build_tv():
    # TV frame
    bb.createBody('box', halfExtent=[0.58, 0.06, 0.36],
                  position=[TV_X, TV_Y+0.03, TV_Z], color='#212121', mass=0)
    # Screen
    bb.createBody('box', halfExtent=[0.54, 0.04, 0.32],
                  position=[TV_X, TV_Y+0.05, TV_Z], color='#1a237e', mass=0)
    # Screen glow (health channel display)
    bb.createBody('box', halfExtent=[0.50, 0.03, 0.28],
                  position=[TV_X, TV_Y+0.07, TV_Z], color='#00bcd4', mass=0)
    # TV stand / arm
    bb.createBody('box', halfExtent=[0.04, 0.06, 0.35],
                  position=[TV_X, TV_Y+0.04, TV_Z-0.70], color='#424242', mass=0)
    bb.createBody('box', halfExtent=[0.16, 0.10, 0.025],
                  position=[TV_X, TV_Y+0.04, TV_Z-1.05], color='#424242', mass=0)
    # Brand logo strip
    bb.createBody('box', halfExtent=[0.10, 0.03, 0.015],
                  position=[TV_X, TV_Y+0.08, TV_Z-0.34], color='#e0e0e0', mass=0)
    print("[TV] Built ✔")


# ═══════════════════════════════════════════════════════════════
#   CLINIC BED + EQUIPMENT
# ═══════════════════════════════════════════════════════════════
def build_bed():
    # Bed frame (clinical white)
    bb.createBody('box', halfExtent=[0.56,1.12,0.22],
                  position=[BED_CX,BED_CY,0.22], color='#e0e0e0', mass=0)
    bb.createBody('box', halfExtent=[0.53,1.07,0.10],
                  position=[BED_CX,BED_CY,0.44], color='#fafafa', mass=0)
    # Pillow
    bb.createBody('box', halfExtent=[0.22,0.24,0.075],
                  position=[BED_CX,BED_CY+1.0,0.575], color='white', mass=0)
    # Blanket
    bb.createBody('box', halfExtent=[0.50,0.68,0.055],
                  position=[BED_CX,BED_CY-0.2,0.505], color='#b3d9f7', mass=0)
    bb.createBody('box', halfExtent=[0.50,0.055,0.06],
                  position=[BED_CX,BED_CY+0.45,0.50], color='#82b9e8', mass=0)
    # Headboard
    bb.createBody('box', halfExtent=[0.57,0.065,0.44],
                  position=[BED_CX,BED_CY+1.54,0.66], color='#bdbdbd', mass=0)
    # Footboard
    bb.createBody('box', halfExtent=[0.57,0.065,0.28],
                  position=[BED_CX,BED_CY-1.18,0.50], color='#bdbdbd', mass=0)
    # Legs
    for lx,ly in [(BED_CX-0.50,BED_CY-1.12),(BED_CX+0.50,BED_CY-1.12),
                  (BED_CX-0.50,BED_CY+1.48),(BED_CX+0.50,BED_CY+1.48)]:
        bb.createBody('box', halfExtent=[0.04,0.04,0.22],
                      position=[lx,ly,0.11], color='#9e9e9e', mass=0)

    # ── Bedside trolley (nurse's equipment cart) ──────────────
    # Cart body
    bb.createBody('box', halfExtent=[0.22,0.32,0.40],
                  position=[0.20, 0.80, 0.40], color='#e8e8e8', mass=0)
    # Cart top
    bb.createBody('box', halfExtent=[0.24,0.34,0.025],
                  position=[0.20, 0.80, 0.825], color='#f5f5f5', mass=0)
    # Tray on cart
    bb.createBody('box', halfExtent=[0.20,0.28,0.015],
                  position=[0.20, 0.80, 0.855], color='#c8d8e8', mass=0)
    # Oxygen meter on tray
    bb.createBody('box', halfExtent=[0.065,0.08,0.055],
                  position=[0.14, 0.72, 0.925], color='#37474f', mass=0)
    bb.createBody('box', halfExtent=[0.055,0.068,0.045],
                  position=[0.14, 0.72, 0.928], color='#4caf50', mass=0)  # screen glow green
    # O2 label sphere
    bb.createBody('sphere', radius=0.025,
                  position=[0.14, 0.64, 0.910], color='#00e676', mass=0)

    # Glucose meter on tray
    bb.createBody('box', halfExtent=[0.055,0.065,0.048],
                  position=[0.26, 0.72, 0.922], color='#1a237e', mass=0)
    bb.createBody('box', halfExtent=[0.045,0.055,0.038],
                  position=[0.26, 0.72, 0.925], color='#ffeb3b', mass=0)   # yellow screen
    bb.createBody('sphere', radius=0.025,
                  position=[0.26, 0.64, 0.910], color='#ffc107', mass=0)

    # Small medicine tray
    bb.createBody('box', halfExtent=[0.08,0.10,0.015],
                  position=[0.20, 0.88, 0.875], color='white', mass=0)
    bb.createBody('sphere', radius=0.016,
                  position=[0.18, 0.84, 0.896], color='#f06292', mass=0)
    bb.createBody('sphere', radius=0.016,
                  position=[0.22, 0.84, 0.896], color='#81d4fa', mass=0)
    bb.createBody('sphere', radius=0.016,
                  position=[0.20, 0.92, 0.896], color='#aed581', mass=0)

    # ── IV Stand ──────────────────────────────────────────────
    bb.createBody('box', halfExtent=[0.024,0.024,0.96],
                  position=[BED_CX+0.82,BED_CY+1.10,0.96], color='#bdbdbd', mass=0)
    bb.createBody('box', halfExtent=[0.22,0.024,0.012],
                  position=[BED_CX+0.82,BED_CY+1.10,1.92], color='#bdbdbd', mass=0)
    bb.createBody('box', halfExtent=[0.09,0.04,0.15],
                  position=[BED_CX+0.82,BED_CY+1.10,1.74], color='#b3e5fc', mass=0)
    bb.createBody('box', halfExtent=[0.005,0.005,0.50],
                  position=[BED_CX+0.70,BED_CY+1.00,1.15], color='#e0e0e0', mass=0)
    bb.createBody('box', halfExtent=[0.24,0.06,0.025],
                  position=[BED_CX+0.82,BED_CY+1.10,0.025], color='#9e9e9e', mass=0)

    # ── Vitals monitor (wall-mounted style) ───────────────────
    bb.createBody('box', halfExtent=[0.28,0.08,0.20],
                  position=[-2.8, BED_CY+0.50, 1.60], color='#263238', mass=0)
    bb.createBody('box', halfExtent=[0.24,0.05,0.17],
                  position=[-2.8, BED_CY+0.55, 1.60], color='#1b5e20', mass=0)
    # ECG green line simulation
    bb.createBody('box', halfExtent=[0.22,0.03,0.005],
                  position=[-2.8, BED_CY+0.57, 1.62], color='#00e676', mass=0)
    # Monitor arm
    bb.createBody('box', halfExtent=[0.03,0.15,0.03],
                  position=[-2.8, BED_CY+0.35, 1.60], color='#546e7a', mass=0)
    # SpO2 display
    bb.createBody('box', halfExtent=[0.08,0.04,0.06],
                  position=[-2.8, BED_CY+0.56, 1.44], color='#00bcd4', mass=0)

    print("[BED] Clinic bed + equipment built ✔")


# ═══════════════════════════════════════════════════════════════
#   PATIENT
# ═══════════════════════════════════════════════════════════════
def build_patient():
    sk = '#f5cba7'
    # Torso
    bb.createBody('box', halfExtent=[0.21,0.42,0.12],
                  position=[BED_CX,BED_CY-0.2,0.59], color='#e3f2fd', mass=0)
    bb.createBody('box', halfExtent=[0.27,0.11,0.09],
                  position=[BED_CX,BED_CY+0.30,0.58], color='#e3f2fd', mass=0)
    # Head
    bb.createBody('sphere', radius=0.178,
                  position=[BED_CX,BED_CY+1.02,0.72], color=sk, mass=0)
    bb.createBody('sphere', radius=0.180,
                  position=[BED_CX,BED_CY+1.04,0.80], color='#5d4037', mass=0)
    # Eyes (closed - resting)
    for ex in [BED_CX-0.08,BED_CX+0.08]:
        bb.createBody('box', halfExtent=[0.036,0.008,0.009],
                      position=[ex,BED_CY+0.89,0.735], color='#4a3728', mass=0)
    bb.createBody('sphere', radius=0.023,
                  position=[BED_CX,BED_CY+0.87,0.705], color='#f0b27a', mass=0)
    bb.createBody('box', halfExtent=[0.042,0.008,0.009],
                  position=[BED_CX,BED_CY+0.86,0.672], color='#e59866', mass=0)
    for ex in [BED_CX-0.19,BED_CX+0.19]:
        bb.createBody('sphere', radius=0.042,
                      position=[ex,BED_CY+1.00,0.72], color=sk, mass=0)
    # Arm with SpO2 sensor (finger clip)
    bb.createBody('box', halfExtent=[0.062,0.29,0.052],
                  position=[BED_CX-0.28,BED_CY+0.40,0.577], color=sk, mass=0)
    bb.createBody('sphere', radius=0.067,
                  position=[BED_CX-0.28,BED_CY+0.10,0.567], color=sk, mass=0)
    # SpO2 finger clip (red)
    bb.createBody('box', halfExtent=[0.040,0.055,0.025],
                  position=[BED_CX-0.28,BED_CY+0.06,0.567], color='#e53935', mass=0)
    # IV tape
    bb.createBody('box', halfExtent=[0.065,0.024,0.055],
                  position=[BED_CX-0.28,BED_CY+0.18,0.567], color='#fff9c4', mass=0)

    print("[PATIENT] Patient built ✔")


# ═══════════════════════════════════════════════════════════════
#   NURSE ROBOT — looks like a nurse
#   White uniform, blue trim, nurse cap, apron
# ═══════════════════════════════════════════════════════════════
class NurseRobot:
    # Resting right-hand local offset
    ARM0_LX =  0.26
    ARM0_LY =  0.00
    ARM0_LZ =  0.30

    def __init__(self):
        self.x    = START_X
        self.y    = START_Y
        self.z    = 0.0
        self.yaw  = 0.0
        self.arm  = 0.0   # right arm extend (check/measure)
        self.larm = 0.0   # left arm extend (clipboard)
        self.wave = 0.0   # wave gesture (0=down,1=up)
        self.nod  = 0.0
        self.wt   = 0.0
        self.ids  = {}
        self.hand_wx = 0.0
        self.hand_wy = 0.0
        self.hand_wz = 0.0
        # Target for arm IK
        self.arm_target = (0.0, 0.0, 0.0)

    def _mk(self, k, shape, **kw):
        kw['mass'] = 0
        self.ids[k] = bb.createBody(shape, **kw)

    def spawn(self):
        d = [0,0,-9]

        # ── UNIFORM BODY ──────────────────────────────────────
        # Main torso (white nurse dress top)
        self._mk('torso', 'box',   halfExtent=[0.155,0.105,0.260], position=d, color='#ffffff')
        # Scrub trousers (light blue)
        self._mk('pants', 'box',   halfExtent=[0.145,0.095,0.155], position=d, color='#bbdefb')
        # Apron panel (front)
        self._mk('apron', 'box',   halfExtent=[0.100,0.020,0.220], position=d, color='#e3f2fd')
        # Red cross badge on chest
        self._mk('crossH','box',   halfExtent=[0.045,0.008,0.012], position=d, color='#e53935')
        self._mk('crossV','box',   halfExtent=[0.012,0.008,0.045], position=d, color='#e53935')
        # Name tag
        self._mk('tag',   'box',   halfExtent=[0.040,0.008,0.018], position=d, color='#1565c0')

        # ── NECK ──────────────────────────────────────────────
        self._mk('neck',  'box',   halfExtent=[0.052,0.052,0.065], position=d, color='#f5cba7')

        # ── HEAD ──────────────────────────────────────────────
        self._mk('head',  'sphere',radius=0.155,                   position=d, color='#f5cba7')
        # Eyes
        self._mk('el',    'sphere',radius=0.038,                   position=d, color='#1a237e')
        self._mk('er',    'sphere',radius=0.038,                   position=d, color='#1a237e')
        # Eye whites
        self._mk('ewl',   'sphere',radius=0.050,                   position=d, color='white')
        self._mk('ewr',   'sphere',radius=0.050,                   position=d, color='white')
        # Eyebrows
        self._mk('ebl',   'box',   halfExtent=[0.040,0.006,0.008], position=d, color='#5d4037')
        self._mk('ebr',   'box',   halfExtent=[0.040,0.006,0.008], position=d, color='#5d4037')
        # Nose
        self._mk('nose',  'sphere',radius=0.020,                   position=d, color='#f0b27a')
        # Smile mouth
        self._mk('mth',   'box',   halfExtent=[0.048,0.007,0.010], position=d, color='#e57373')
        # Ears
        self._mk('earl',  'sphere',radius=0.036,                   position=d, color='#f5cba7')
        self._mk('earr',  'sphere',radius=0.036,                   position=d, color='#f5cba7')

        # ── NURSE CAP ─────────────────────────────────────────
        self._mk('cap',   'box',   halfExtent=[0.145,0.105,0.038], position=d, color='white')
        self._mk('cap2',  'box',   halfExtent=[0.100,0.075,0.055], position=d, color='white')
        # Blue stripe on cap
        self._mk('capst', 'box',   halfExtent=[0.148,0.107,0.010], position=d, color='#1565c0')
        # Red cross on cap
        self._mk('capcH', 'box',   halfExtent=[0.030,0.006,0.008], position=d, color='#e53935')
        self._mk('capcV', 'box',   halfExtent=[0.008,0.006,0.030], position=d, color='#e53935')

        # ── LEFT ARM (holds clipboard/device) ─────────────────
        self._mk('lau',   'box',   halfExtent=[0.048,0.048,0.155], position=d, color='#ffffff')
        self._mk('lal',   'box',   halfExtent=[0.040,0.040,0.125], position=d, color='#f5cba7')
        self._mk('lah',   'sphere',radius=0.050,                   position=d, color='#f5cba7')
        # Clipboard in left hand
        self._mk('clip',  'box',   halfExtent=[0.075,0.015,0.100], position=d, color='#8d6e63')
        self._mk('paper', 'box',   halfExtent=[0.065,0.010,0.088], position=d, color='white')
        # Lines on paper
        self._mk('line1', 'box',   halfExtent=[0.050,0.005,0.006], position=d, color='#bdbdbd')
        self._mk('line2', 'box',   halfExtent=[0.050,0.005,0.006], position=d, color='#bdbdbd')
        self._mk('line3', 'box',   halfExtent=[0.050,0.005,0.006], position=d, color='#bdbdbd')

        # ── RIGHT ARM (O2/glucose probe) ──────────────────────
        self._mk('rau',   'box',   halfExtent=[0.048,0.048,0.155], position=d, color='#ffffff')
        self._mk('ral',   'box',   halfExtent=[0.040,0.040,0.125], position=d, color='#f5cba7')
        self._mk('rah',   'sphere',radius=0.050,                   position=d, color='#f5cba7')
        # Medical probe in right hand
        self._mk('probe', 'box',   halfExtent=[0.022,0.022,0.075], position=d, color='#37474f')
        self._mk('probeL','sphere',radius=0.028,                   position=d, color='#00e676')

        # ── LEGS ──────────────────────────────────────────────
        self._mk('llt',   'box',   halfExtent=[0.058,0.058,0.160], position=d, color='#bbdefb')
        self._mk('lls',   'box',   halfExtent=[0.048,0.048,0.120], position=d, color='#bbdefb')
        self._mk('llf',   'box',   halfExtent=[0.070,0.042,0.030], position=d, color='white')
        self._mk('rlt',   'box',   halfExtent=[0.058,0.058,0.160], position=d, color='#bbdefb')
        self._mk('rls',   'box',   halfExtent=[0.048,0.048,0.120], position=d, color='#bbdefb')
        self._mk('rlf',   'box',   halfExtent=[0.070,0.042,0.030], position=d, color='white')

        self.update()
        print("[NURSE ROBOT] Spawned ✔")

    def _w(self, lx, ly, lz):
        cy, sy = math.cos(self.yaw), math.sin(self.yaw)
        return (self.x+cy*lx-sy*ly, self.y+sy*lx+cy*ly, self.z+lz)

    def _p(self, key, lx, ly, lz, rx=0, ry=0):
        wx,wy,wz = self._w(lx,ly,lz)
        place(self.ids[key], wx,wy,wz, rx,ry, self.yaw)

    def update(self):
        a   = self.arm
        la  = self.larm
        w   = self.wave
        nd  = self.nod
        wt  = self.wt

        LL = math.sin(wt)*0.06
        RL = math.sin(wt+math.pi)*0.06

        # ── Body ──────────────────────────────────────────────
        self._p('torso', 0, 0,      0.740)
        self._p('pants', 0, 0,      0.390)
        self._p('apron', 0,-0.108,  0.730)
        self._p('crossH',0,-0.118,  0.840)
        self._p('crossV',0,-0.118,  0.840)
        self._p('tag',   0,-0.118,  0.800)

        # ── Neck + Head ───────────────────────────────────────
        self._p('neck', 0, 0, 1.018)
        hx,hy,hz = self._w(0,0,1.210)
        place(self.ids['head'], hx,hy,hz, nd,0,self.yaw)

        # ── Nurse cap (on top of head) ────────────────────────
        cx,cy_,cz = self._w(0,0,1.395)
        place(self.ids['cap'],  cx,cy_,cz,       nd,0,self.yaw)
        place(self.ids['cap2'], cx,cy_,cz+0.050, nd,0,self.yaw)
        place(self.ids['capst'],cx,cy_,cz-0.020, nd,0,self.yaw)
        place(self.ids['capcH'],cx,cy_,cz+0.042, nd,0,self.yaw)
        place(self.ids['capcV'],cx,cy_,cz+0.042, nd,0,self.yaw)

        # ── Face features (front = local -Y) ──────────────────
        self._p('ewl', -0.062,-0.125, 1.228, nd)
        self._p('ewr',  0.062,-0.125, 1.228, nd)
        self._p('el',  -0.062,-0.138, 1.228, nd)
        self._p('er',   0.062,-0.138, 1.228, nd)
        self._p('ebl', -0.062,-0.138, 1.252, nd)
        self._p('ebr',  0.062,-0.138, 1.252, nd)
        self._p('nose', 0,    -0.148, 1.198, nd)
        self._p('mth',  0,    -0.145, 1.170, nd)
        self._p('earl',-0.194, 0.010, 1.210, nd)
        self._p('earr', 0.194, 0.010, 1.210, nd)

        # ── LEFT ARM — holds clipboard, slight raised when active ─
        wzu = 0.810 + la*0.120
        wzl = 0.510 + la*0.200
        wzh = 0.330 + la*0.280
        lly = -la*0.150
        self._p('lau', -0.230, 0,    wzu)
        self._p('lal', -0.230, lly,  wzl)
        self._p('lah', -0.230, lly,  wzh)
        # Clipboard follows left hand
        clx,cly,clz = self._w(-0.230, lly-0.016, wzh+0.005)
        place(self.ids['clip'], clx,cly,clz, 0,0,self.yaw)
        place(self.ids['paper'],clx,cly,clz, 0,0,self.yaw)
        place(self.ids['line1'],clx,cly,clz+0.040, 0,0,self.yaw)
        place(self.ids['line2'],clx,cly,clz+0.010, 0,0,self.yaw)
        place(self.ids['line3'],clx,cly,clz-0.020, 0,0,self.yaw)

        # ── RIGHT ARM — IK toward arm_target ──────────────────
        tx, ty, tz = self.arm_target
        cy_r, sy_r = math.cos(self.yaw), math.sin(self.yaw)
        dm_x = tx - self.x
        dm_y = ty - self.y
        dz   = tz - self.z
        lx1  =  cy_r*dm_x + sy_r*dm_y
        ly1  = -sy_r*dm_x + cy_r*dm_y
        lz1  =  dz

        lx0, ly0, lz0 = self.ARM0_LX, self.ARM0_LY, self.ARM0_LZ
        t_  = ease(a)
        hlx = lerp(lx0, lx1, t_)
        hly = lerp(ly0, ly1, t_)
        hlz = lerp(lz0, lz1, t_)

        self._p('rau', 0.230, lerp(0, ly1*0.28, t_), lerp(0.810, lz1*0.72, t_), -t_*0.85)
        self._p('ral', lerp(0.230,hlx*0.82,t_), lerp(0,hly*0.62,t_), lerp(0.510,hlz*0.78,t_))
        hwx,hwy,hwz = self._w(hlx,hly,hlz)
        place(self.ids['rah'],   hwx,hwy,hwz, 0,0,self.yaw)
        place(self.ids['probe'], hwx,hwy,hwz, 0,0,self.yaw)
        place(self.ids['probeL'],hwx,hwy,hwz+0.082, 0,0,self.yaw)
        self.hand_wx, self.hand_wy, self.hand_wz = hwx,hwy,hwz

        # ── LEGS ──────────────────────────────────────────────
        self._p('llt',-0.10, LL*0.5,  0.270)
        self._p('lls',-0.10, LL*0.25, 0.090)
        self._p('llf',-0.10, 0,       0.022)
        self._p('rlt', 0.10, RL*0.5,  0.270)
        self._p('rls', 0.10, RL*0.25, 0.090)
        self._p('rlf', 0.10, 0,       0.022)

    def set_target(self, tx, ty, tz):
        """Set where right arm probe tip should reach."""
        self.arm_target = (tx, ty, tz)

    def set_pose(self, x, y, yaw):
        self.x, self.y, self.yaw = x, y, yaw
        self.update()


# ═══════════════════════════════════════════════════════════════
#   HUD / STATUS
# ═══════════════════════════════════════════════════════════════
_sid=[None]; _rid=[None]; _vit=[None]

def status(msg, col='#a5d6a7'):
    if _sid[0]: bb.removeDebugObject(_sid[0])
    _sid[0] = bb.createDebugText(f"▶  {msg}", (0,-5.2,0.50),
        bb.getQuaternionFromEuler([0,0,0]), color=col, size=0.20)
    print(f"   >> {msg}")

def set_run(n):
    if _rid[0]: bb.removeDebugObject(_rid[0])
    _rid[0] = bb.createDebugText(f"NURSE-BOT v1  |  Clinic Room 204  |  Run #{n}",
        (0,0,3.40), bb.getQuaternionFromEuler([0,0,0]),
        color='#00e5ff', size=0.22)

def show_vitals(o2, glucose):
    if _vit[0]: bb.removeDebugObject(_vit[0])
    o2_col  = '#00e676' if o2 >= 95 else '#ff5252'
    gl_col  = '#ffe082' if 70<=glucose<=140 else '#ff5252'
    _vit[0] = bb.createDebugText(
        f"SpO2: {o2}%  |  Glucose: {glucose} mg/dL",
        (0, 0, 3.10),
        bb.getQuaternionFromEuler([0,0,0]),
        color=o2_col, size=0.20)
    print(f"   [VITALS] SpO2={o2}%  Glucose={glucose} mg/dL")

def clear_vitals():
    if _vit[0]: bb.removeDebugObject(_vit[0])
    _vit[0] = None


# ═══════════════════════════════════════════════════════════════
#   MOTION HELPERS
# ═══════════════════════════════════════════════════════════════
def _sync(nurse, carry=False):
    pass   # probe always follows hand in update()

def walk_to(nurse, tx, ty, spd=1.6):
    sx,sy = nurse.x, nurse.y
    d = math.sqrt((tx-sx)**2+(ty-sy)**2)
    if d < 0.02: return
    tyaw = face_yaw(sx,sy,tx,ty)
    turn_to(nurse, tyaw, steps=20)
    steps = max(1, int(d/(spd*TICK)))
    for i in range(steps+1):
        t = ease(i/steps)
        nurse.x = lerp(sx,tx,t)
        nurse.y = lerp(sy,ty,t)
        nurse.wt += 0.18
        nurse.update()
        time.sleep(TICK)
    nurse.x,nurse.y = tx,ty
    nurse.wt=0.0; nurse.update()

def turn_to(nurse, tyaw, steps=28):
    start = nurse.yaw
    diff  = norm_angle(tyaw-start)
    if abs(diff)<0.01: return
    for i in range(steps+1):
        nurse.yaw = start+diff*ease(i/steps)
        nurse.update()
        time.sleep(TICK)

def move_arm(nurse, target, steps=40):
    start = nurse.arm
    for i in range(steps+1):
        nurse.arm = lerp(start,target,ease(i/steps))
        nurse.update()
        time.sleep(TICK)

def move_larm(nurse, target, steps=32):
    start = nurse.larm
    for i in range(steps+1):
        nurse.larm = lerp(start,target,ease(i/steps))
        nurse.update()
        time.sleep(TICK)

def do_nod(nurse, amp=0.28, reps=2):
    steps=20
    for _ in range(reps):
        for i in range(steps+1):
            nurse.nod = amp*math.sin(math.pi*i/steps)
            nurse.update()
            time.sleep(TICK)
    nurse.nod=0.0; nurse.update()

def do_wave(nurse, reps=3):
    """Wave right hand (arm goes up/down repeatedly for goodbye)."""
    # Temporarily use wave as arm oscillation
    for _ in range(reps):
        move_arm(nurse, 0.70, steps=15)
        time.sleep(0.08)
        move_arm(nurse, 0.45, steps=12)
        time.sleep(0.06)
    move_arm(nurse, 0.0, steps=20)

def do_spin(nurse, rounds=1, steps=58):
    start=nurse.yaw
    for i in range(steps+1):
        nurse.yaw=start+rounds*2*math.pi*ease(i/steps)
        nurse.update()
        time.sleep(TICK)
    nurse.yaw=start; nurse.update()

def curtain_pause(steps=38, tick_mul=1.8):
    """Used inside curtain animation — just waits proportionally."""
    time.sleep(steps*TICK*tick_mul)


# ═══════════════════════════════════════════════════════════════
#   MAIN SEQUENCE
# ═══════════════════════════════════════════════════════════════

def run_sequence(nurse, window, run_no):
    set_run(run_no)
    nurse.arm=0.0; nurse.larm=0.0; nurse.nod=0.0; nurse.wt=0.0
    nurse.set_target(CHECK_X+1.2, CHECK_Y-0.1, 0.70)  # default resting target
    nurse.set_pose(START_X, START_Y, 0.0)
    clear_vitals()

    status("NURSE-BOT warming up...", '#80deea')
    time.sleep(0.8)

    # ── 1. Walk to patient bedside ────────────────────────────
    status("Walking to patient bedside 🚶", '#80deea')
    walk_to(nurse, CHECK_X, CHECK_Y, spd=1.6)

    # Face patient precisely
    yaw_pt = face_yaw(CHECK_X, CHECK_Y,
                      BED_CX+PATIENT_HEAD[0]-BED_CX,
                      BED_CY+PATIENT_HEAD[1]-BED_CY)
    # Actually face toward patient head world pos
    yaw_pt = face_yaw(nurse.x, nurse.y, PATIENT_HEAD[0], PATIENT_HEAD[1])
    turn_to(nurse, yaw_pt, steps=25)
    time.sleep(0.25)

    # ── 2. Greet patient ─────────────────────────────────────
    status("Hello! Good morning, I am your nurse robot 👋", '#fff9c4')
    do_nod(nurse, amp=0.22, reps=2)
    move_larm(nurse, 0.60, steps=20)   # raise clipboard
    time.sleep(0.30)

    # ── 3. Open curtains (morning light check) ────────────────
    status("Opening curtains for morning light 🌅", '#ffe082')
    # Walk to curtain area
    walk_to(nurse, WIN_X-0.55, WIN_CY, spd=1.5)
    turn_to(nurse, face_yaw(nurse.x, nurse.y, WIN_X, WIN_CY), steps=20)
    time.sleep(0.20)

    # Raise arm toward curtain cord
    nurse.set_target(WIN_X-0.18, WIN_CY-0.70, WIN_Z+0.50)
    move_arm(nurse, 0.90, steps=30)
    time.sleep(0.30)

    # Animate curtains opening
    window.animate_open(steps=45)

    move_arm(nurse, 0.0, steps=22)
    time.sleep(0.25)

    # Return to patient
    walk_to(nurse, CHECK_X, CHECK_Y, spd=1.6)
    turn_to(nurse, face_yaw(nurse.x, nurse.y,
                            PATIENT_HEAD[0], PATIENT_HEAD[1]), steps=22)
    time.sleep(0.20)

    # ── 4. CHECK OXYGEN LEVEL (SpO2) ─────────────────────────
    status("Checking SpO2 (Oxygen Level) 🩺", '#80deea')

    # Target = patient's finger clip position
    O2_TARGET_X = BED_CX - 0.28
    O2_TARGET_Y = BED_CY + 0.10 + BED_CY*0  # actual world Y
    # Patient arm is at (BED_CX-0.28, BED_CY+0.10, 0.567)
    O2_TARGET_X = BED_CX - 0.28
    O2_TARGET_Y = BED_CY + 0.06
    O2_TARGET_Z = 0.567

    nurse.set_target(O2_TARGET_X, O2_TARGET_Y, O2_TARGET_Z)
    move_arm(nurse, 1.0, steps=42)
    time.sleep(0.40)

    # Simulate reading (hold 2 sec with visual pulse)
    status("Reading oxygen sensor... ⏳", '#b3e5fc')
    for pulse in range(3):
        nurse.ids['probeL']  # just update
        nurse.nod = 0.06; nurse.update(); time.sleep(0.30)
        nurse.nod = 0.0;  nurse.update(); time.sleep(0.30)

    # Generate simulated O2 reading
    o2_val = random.randint(94, 99)
    show_vitals(o2_val, 0)
    status(f"SpO2 = {o2_val}% — {'NORMAL ✔' if o2_val>=95 else 'LOW ⚠ Alerting!'}", '#00e676' if o2_val>=95 else '#ff5252')

    # Note on clipboard
    move_larm(nurse, 0.80, steps=18)
    time.sleep(0.35)
    move_larm(nurse, 0.50, steps=14)

    move_arm(nurse, 0.0, steps=28)
    time.sleep(0.35)

    # ── 5. CHECK GLUCOSE LEVEL ────────────────────────────────
    status("Checking Glucose Level 🩸", '#ffe082')

    # Glucose sensor tip on patient arm (slightly different spot)
    GL_TARGET_X = BED_CX - 0.28
    GL_TARGET_Y = BED_CY + 0.22
    GL_TARGET_Z = 0.577

    nurse.set_target(GL_TARGET_X, GL_TARGET_Y, GL_TARGET_Z)
    move_arm(nurse, 1.0, steps=40)
    time.sleep(0.40)

    status("Reading glucose sensor... ⏳", '#fff9c4')
    for pulse in range(3):
        nurse.nod = 0.06; nurse.update(); time.sleep(0.28)
        nurse.nod = 0.0;  nurse.update(); time.sleep(0.28)

    gl_val = random.randint(88, 145)
    show_vitals(o2_val, gl_val)
    normal_gl = 70 <= gl_val <= 140
    status(f"Glucose = {gl_val} mg/dL — {'NORMAL ✔' if normal_gl else 'ABNORMAL ⚠ Check diet!'}", '#aed581' if normal_gl else '#ff9800')

    move_larm(nurse, 0.85, steps=16)
    time.sleep(0.35)
    move_larm(nurse, 0.55, steps=14)

    move_arm(nurse, 0.0, steps=28)
    time.sleep(0.35)

    # ── 6. Record results (writes on clipboard) ───────────────
    status("Recording vitals on chart 📋", '#b2dfdb')
    move_larm(nurse, 1.0, steps=22)
    do_nod(nurse, amp=0.15, reps=3)    # looks down at chart while writing
    time.sleep(0.40)
    move_larm(nurse, 0.60, steps=16)
    time.sleep(0.30)

    # ── 7. Check if curtains should close (if O2 low, close for rest) ─
    if o2_val < 96:
        status("O2 slightly low — closing curtains for patient rest 😴", '#b3e5fc')
        walk_to(nurse, WIN_X-0.55, WIN_CY, spd=1.5)
        turn_to(nurse, face_yaw(nurse.x, nurse.y, WIN_X, WIN_CY), steps=20)
        nurse.set_target(WIN_X-0.18, WIN_CY-0.70, WIN_Z+0.50)
        move_arm(nurse, 0.85, steps=28)
        time.sleep(0.25)
        window.animate_close(steps=45)
        move_arm(nurse, 0.0, steps=20)
        time.sleep(0.25)
        walk_to(nurse, CHECK_X, CHECK_Y, spd=1.6)
        turn_to(nurse, face_yaw(nurse.x, nurse.y,
                                PATIENT_HEAD[0], PATIENT_HEAD[1]), steps=22)
    else:
        status("Vitals normal — curtains stay open 🌤", '#a5d6a7')
        time.sleep(0.40)

    # ── 8. Goodbye to patient ─────────────────────────────────
    status("Goodbye! Rest well. I'll check again soon 👋", '#fff9c4')
    nurse.set_target(CHECK_X+1.2, CHECK_Y, 0.70)
    do_nod(nurse, amp=0.28, reps=1)
    do_wave(nurse, reps=3)
    time.sleep(0.25)

    # ── 9. Bow ───────────────────────────────────────────────
    status("Bowing goodbye 🙏", '#b2dfdb')
    do_nod(nurse, amp=0.52, reps=1)
    time.sleep(0.20)

    # ── 10. Spin + return ────────────────────────────────────
    status("🎉 Health check complete!", '#80cbc4')
    do_spin(nurse, rounds=1, steps=56)
    move_larm(nurse, 0.0, steps=18)
    time.sleep(0.35)

    status("Returning to nurse station...", '#b0bec5')
    walk_to(nurse, START_X, START_Y, spd=1.9)
    turn_to(nurse, 0.0, steps=18)
    status("Standby — next check in 60s ✔", '#80deea')
    time.sleep(1.0)


# ═══════════════════════════════════════════════════════════════
#   MAIN
# ═══════════════════════════════════════════════════════════════
print("="*58)
print("  BROWSERBOTICS — NURSE ROBOT v6")
print("  Clinic: bed + TV + curtain window")
print("  Tasks: SpO2 check, Glucose check, Curtain open/close")
print("="*58)

build_clinic()
build_tv()
build_bed()
build_patient()

window = Window()

nurse = NurseRobot()
nurse.set_target(CHECK_X+1.2, CHECK_Y, 0.70)
nurse.spawn()

# Static HUD
bb.createDebugText("✚  NURSE-BOT v1  |  Smart Clinic 204",
    (0,0,3.70), bb.getQuaternionFromEuler([0,0,0]),
    color='#00e5ff', size=0.26)
bb.createDebugText("Tasks: Open Curtains → SpO2 Check → Glucose Check → Record → Goodbye",
    (0,0,3.50), bb.getQuaternionFromEuler([0,0,0]),
    color='white', size=0.15)

status("World loaded — starting in 2s...", '#80deea')
time.sleep(2.0)

run_no = 0
while True:
    run_no += 1
    print(f"\n{'='*50}\n  HEALTH CHECK RUN #{run_no}\n{'='*50}")
    run_sequence(nurse, window, run_no)
    print(f"  Run #{run_no} done. Restarting in 4s...")
    time.sleep(4.0)
