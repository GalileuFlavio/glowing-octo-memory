# glowing-octo-memory
driver de dispositivo no Linux usando a linguagem C
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/uaccess.h>

#define DEVICE_NAME "meu_driver"
#define BUFFER_SIZE 1024

static int major_number;
static char buffer[BUFFER_SIZE];
static int buffer_len;

static int device_open(struct inode *inode, struct file *file)
{
    printk(KERN_INFO "meu_driver: Dispositivo aberto\n");
    return 0;
}

static int device_release(struct inode *inode, struct file *file)
{
    printk(KERN_INFO "meu_driver: Dispositivo fechado\n");
    return 0;
}

static ssize_t device_read(struct file *filp, char __user *buf, size_t len, loff_t *offset)
{
    int bytes_read = 0;
    if (*offset >= buffer_len)
        return 0;
    
    bytes_read = len > (buffer_len - *offset) ? (buffer_len - *offset) : len;
    if (copy_to_user(buf, buffer + *offset, bytes_read) != 0)
        return -EFAULT;
    
    *offset += bytes_read;
    return bytes_read;
}

static ssize_t device_write(struct file *filp, const char __user *buf, size_t len, loff_t *offset)
{
    int bytes_written = 0;
    if (len > BUFFER_SIZE)
        return -EFAULT;
    
    if (copy_from_user(buffer, buf, len) != 0)
        return -EFAULT;
    
    buffer_len = len;
    *offset += len;
    return len;
}

static struct file_operations fops = {
    .open = device_open,
    .release = device_release,
    .read = device_read,
    .write = device_write,
};

static int __init driver_init(void)
{
    major_number = register_chrdev(0, DEVICE_NAME, &fops);
    if (major_number < 0) {
        printk(KERN_ALERT "meu_driver: Falha ao registrar o dispositivo\n");
        return major_number;
    }
    printk(KERN_INFO "meu_driver: Dispositivo registrado com sucesso. Major number: %d\n", major_number);
    return 0;
}

static void __exit driver_exit(void)
{
    unregister_chrdev(major_number, DEVICE_NAME);
    printk(KERN_INFO "meu_driver: Dispositivo removido\n");
}

module_init(driver_init);
module_exit(driver_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Seu Nome");
MODULE_DESCRIPTION("Exemplo de driver de dispositivo no Linux");
