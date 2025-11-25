import hid
import time

VENDOR_ID = 0x10C4
PRODUCT_ID = 0xEA90

# Report IDs laut AN495
REPORT_GPIO_CONFIG = 0x02   # Get/Set GPIO Configuration (Feature)
REPORT_GPIO_GET    = 0x03   # Get GPIO (Feature)
REPORT_GPIO_SET    = 0x04   # Set GPIO (Feature)


def find_cp2112():
    for d in hid.enumerate():
        if d['vendor_id'] == VENDOR_ID and d['product_id'] == PRODUCT_ID:
            return d
    return None


def open_device():
    info = find_cp2112()
    if info is None:
        raise RuntimeError("CP2112 not found (VID=10C4 PID=EA90).")
    dev = hid.device()
    # unter Windows safer: über path öffnen
    dev.open_path(info['path'])
    dev.set_nonblocking(0)
    return dev


def gpio0_as_pushpull_output(dev):
    """
    Set GPIO0 as Output, Push-Pull, keine Special-Funktion.
    Report 0x02 (Get/Set GPIO Configuration)

    Offset 1: Direction bits  (1 = Output)
    Offset 2: Push-Pull bits  (1 = Push-Pull)
    Offset 3: Special bits    (0 = alles aus)
    Offset 4: Clock Divider   (0 = 48 MHz, hier egal)
    """

    direction = 0x01   # Bit0 = GPIO0 als Output
    pushpull  = 0x01   # Bit0 = GPIO0 Push-Pull
    special   = 0x00   # keine TX/RX/Clock-Specials
    clkdiv    = 0x00

    report = [
        REPORT_GPIO_CONFIG,
        direction,
        pushpull,
        special,
        clkdiv
    ]
    dev.send_feature_report(report)


def gpio0_set(dev, value: bool):
    """
    Set GPIO0 High/Low.
    Report 0x04 (Set GPIO Values)

    Offset 1: Latch Value (Bit0 = GPIO0)
    Offset 2: Latch Mask  (Bit0 = 1 -> GPIO0 wird geändert)
    """

    latch_value = 0x01 if value else 0x00
    latch_mask  = 0x01  # nur GPIO0

    report = [
        REPORT_GPIO_SET,
        latch_value,
        latch_mask
    ]
    dev.send_feature_report(report)


def main():
    dev = open_device()
    print("Connected to CP2112.")

    gpio0_as_pushpull_output(dev)
    print("Configured GPIO0 as Output / Push-Pull.")

    state = False
    while True:
        gpio0_set(dev, state)
        # Debug-Ausgabe, kannst du wegmachen
        # print("GPIO0 ->", "HIGH" if state else "LOW")
        state = not state
        time.sleep(0.5)


if __name__ == "__main__":
    main()
